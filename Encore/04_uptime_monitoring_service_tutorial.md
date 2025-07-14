# Encore Uptime Monitoring Service Tutorial

This guide walks you through building an uptime monitoring service using Encore, TypeScript, PostgreSQL, and Slack notifications. It is structured to be sequential and ready for quick reference.

---

## 1. Create a New Encore App

1. Install Encore if you haven't:
    ```
    brew install encoredev/tap/encore
    ```
2. Create a new Encore project:
    ```
    encore app create
    ```
3. Run the app locally:
    ```
    encore run
    ```

---

## 2. Create the `monitor` Service

### 2.1. Setup

- Create a directory:  
  `monitor`
- Add `encore.service.ts`:
    ```ts
    import { Service } from "encore.dev/service";
    export default new Service("monitor");
    ```

### 2.2. Create the `ping` API

- Create `ping.ts`:
    ```ts
    // Service monitor checks if a website is up or down.
    import { api } from "encore.dev/api";

    export interface PingParams { url: string; }
    export interface PingResponse { up: boolean; }

    export const ping = api<PingParams, PingResponse>(
      { expose: true, path: "/ping/:url", method: "GET" },
      async ({ url }) => {
        if (!url.startsWith("http:") && !url.startsWith("https:")) {
          url = "https://" + url;
        }
        try {
          const resp = await fetch(url, { method: "GET" });
          const up = resp.status >= 200 && resp.status < 300;
          return { up };
        } catch (err) {
          return { up: false };
        }
      }
    );
    ```

### 2.3. Add Tests

- Create `ping.test.ts`:
    ```ts
    import { describe, expect, test } from "vitest";
    import { ping } from "./ping";

    describe("ping", () => {
      test.each([
        { site: "google.com", expected: true },
        { site: "https://encore.dev", expected: true },
        { site: "https://not-a-real-site.xyz", expected: false },
        { site: "invalid://scheme", expected: false },
      ])(
        `should verify that $site is ${"$expected" ? "up" : "down"}`,
        async ({ site, expected }) => {
          const resp = await ping({ url: site });
          expect(resp.up).toBe(expected);
        },
      );
    });
    ```
- Install Vitest:
    ```
    pnpm install vitest
    ```
- Add to `package.json`:
    ```json
    "scripts": { "test": "vitest" }
    ```
- Run the tests:
    ```
    encore test
    ```

---

## 3. Create the `site` Service

### 3.1. Setup

- Create directory:  
  `site`
- Add `encore.service.ts`:
    ```ts
    import { Service } from "encore.dev/service";
    export default new Service("site");
    ```

### 3.2. Set Database Migrations

- Create migration file `site/migrations/1_create_tables_up.sql`:
    ```sql
    CREATE TABLE site (
        id SERIAL PRIMARY KEY,
        url TEXT NOT NULL UNIQUE
    );
    ```

### 3.3. Add ORM

- Install knex and pg:
    ```
    pnpm i knex pg
    ```

### 3.4. Add Site APIs (`site.ts`)

- Create `site.ts`:
    ```ts
    import { api } from "encore.dev/api";
    import { SQLDatabase } from "encore.dev/storage/sqldb";
    import knex from "knex";

    export interface Site { id: number; url: string; }
    export interface AddParams { url: string; }

    export const add = api(
      { expose: true, method: "POST", path: "/site" },
      async (params: AddParams): Promise<Site> => {
        const site = (await Sites().insert({ url: params.url }, "*"))[0];
        return site;
      },
    );

    export const get = api(
      { expose: true, method: "GET", path: "/site/:id", auth: false },
      async ({ id }: { id: number }): Promise<Site> => {
        const site = await Sites().where("id", id).first();
        return site ?? Promise.reject(new Error("site not found"));
      },
    );

    export const del = api(
      { expose: true, method: "DELETE", path: "/site/:id" },
      async ({ id }: { id: number }): Promise<void> => {
        await Sites().where("id", id).delete();
      },
    );

    export interface ListResponse { sites: Site[]; }
    export const list = api(
      { expose: true, method: "GET", path: "/site" },
      async (): Promise<ListResponse> => {
        const sites = await Sites().select();
        return { sites };
      },
    );

    const SiteDB = new SQLDatabase("site", {
      migrations: "./migrations",
    });

    const orm = knex({
      client: "pg",
      connection: SiteDB.connectionString,
    });

    const Sites = () => orm<Site>("site");
    ```

### 3.5. Restart the Service

- Restart:
    ```
    encore run
    ```

---

## 4. Add the `check` Table and Monitoring APIs

### 4.1. Migration

- Create `monitor/migrations/1_create_tables.up.sql`:
    ```sql
    CREATE TABLE checks (
        id BIGSERIAL PRIMARY KEY,
        site_id BIGINT NOT NULL,
        up BOOLEAN NOT NULL,
        checked_at TIMESTAMP WITH TIME ZONE NOT NULL
    );
    ```

### 4.2. Add Check APIs (`monitor/check.ts`)

- Example implementation:
    ```ts
    import { api } from "encore.dev/api";
    import { SQLDatabase } from "encore.dev/storage/sqldb";
    import { ping } from "./ping";
    import { site } from "~encore/clients";
    import { Site } from "../site/site";

    // Check checks a single site.
    export const check = api(
      { expose: true, method: "POST", path: "/check/:siteID" },
      async (p: { siteID: number }): Promise<{ up: boolean }> => {
        const s = await site.get({ id: p.siteID });
        return doCheck(s);
      },
    );

    async function doCheck(site: Site): Promise<{ up: boolean }> {
      const { up } = await ping({ url: site.url });
      await MonitorDB.exec`
          INSERT INTO checks (site_id, up, checked_at)
          VALUES (${site.id}, ${up}, NOW())
      `;
      return { up };
    }

    export const MonitorDB = new SQLDatabase("monitor", {
      migrations: "./migrations",
    });
    ```

### 4.3. Add `checkAll` API

- Add to `monitor/check.ts`:
    ```ts
    export const checkAll = api(
      { expose: true, method: "POST", path: "/check-all" },
      async (): Promise<void> => {
        const sites = await site.list();
        await Promise.all(sites.sites.map(doCheck));
      },
    );
    ```

### 4.4. Add Cron Job

- Add to `monitor/check.ts`:
    ```ts
    import { CronJob } from "encore.dev/cron";
    const cronJob = new CronJob("check-all", {
      title: "Check all sites",
      every: "1h",
      endpoint: checkAll,
    });
    ```

### 4.5. Add Status API

- Create `monitor/status.ts`:
    ```ts
    import { api } from "encore.dev/api";
    import { MonitorDB } from "./check";

    interface SiteStatus { id: number; up: boolean; checkedAt: string; }
    interface StatusResponse { sites: SiteStatus[]; }

    export const status = api(
      { expose: true, path: "/status", method: "GET" },
      async (): Promise<StatusResponse> => {
        const rows = await MonitorDB.query`
          SELECT DISTINCT ON (site_id) site_id, up, checked_at
          FROM checks
          ORDER BY site_id, checked_at DESC
        `;
        const results: SiteStatus[] = [];
        for await (const row of rows) {
          results.push({
            id: row.site_id,
            up: row.up,
            checkedAt: row.checked_at,
          });
        }
        return { sites: results };
      },
    );
    ```

---

## 5. Set Up Pub/Sub for Uptime Transitions

### 5.1. Define Pub/Sub Topic

- In `monitor/check.ts`:
    ```ts
    import { Subscription, Topic } from "encore.dev/pubsub";
    import { Site } from "../site/site";

    export interface TransitionEvent {
      site: Site;
      up: boolean;
    }

    export const TransitionTopic = new Topic<TransitionEvent>("uptime-transition", {
      deliveryGuarantee: "at-least-once",
    });
    ```

### 5.2. Publish Transition Events

- Update `doCheck` and add helper:
    ```ts
    async function getPreviousMeasurement(siteID: number): Promise<boolean> {
      const row = await MonitorDB.queryRow`
          SELECT up
          FROM checks
          WHERE site_id = ${siteID}
          ORDER BY checked_at DESC
          LIMIT 1
      `;
      return row?.up ?? true;
    }

    async function doCheck(site: Site): Promise<{ up: boolean }> {
      const { up } = await ping({ url: site.url });

      const wasUp = await getPreviousMeasurement(site.id);
      if (up !== wasUp) {
        await TransitionTopic.publish({ site, up });
      }

      await MonitorDB.exec`
          INSERT INTO checks (site_id, up, checked_at)
          VALUES (${site.id}, ${up}, NOW())
      `;
      return { up };
    }
    ```

---

## 6. Send Slack Notifications

### 6.1. Create the `slack` Service

- Create directory `slack` and `encore.service.ts`:
    ```ts
    import { Service } from "encore.dev/service";
    export default new Service("slack");
    ```

### 6.2. Implement Notification API

- Add `slack.ts`:
    ```ts
    import { api } from "encore.dev/api";
    import { secret } from "encore.dev/config";
    import log from "encore.dev/log";

    export interface NotifyParams { text: string; }

    export const notify = api<NotifyParams>({}, async ({ text }) => {
      const url = webhookURL();
      if (!url) {
        log.info("no slack webhook url defined, skipping slack notification");
        return;
      }
      const resp = await fetch(url, {
        method: "POST",
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ content: text }),
      });
      if (resp.status >= 400) {
        const body = await resp.text();
        throw new Error(`slack notification failed: ${resp.status}: ${body}`);
      }
    });

    const webhookURL = secret("SlackWebhookURL");
    ```

### 6.3. Set Slack Webhook Secret

- Set the secret:
    ```
    encore secret set --type dev,local,pr SlackWebhookURL
    ```

### 6.4. Test Slack Notification

- Test manually:
    ```
    curl 'http://localhost:4000/slack.notify' -d '{"text": "Testing Slack webhook"}'
    ```

### 6.5. Subscribe to Pub/Sub Transitions

- Add to `slack/slack.ts` or a new file:
    ```ts
    import { Subscription } from "encore.dev/pubsub";
    import { TransitionTopic } from "../monitor/check";
    import { notify } from "./slack";

    const _ = new Subscription(TransitionTopic, "slack-notification", {
      handler: async (event) => {
        const text = `*${event.site.url} is ${event.up ? "back up." : "down!"}*`;
        await notify({ text });
      },
    });
    ```

---

## 7. Deploy

- Deploy to Encore Cloud or your target environment:
    ```
    encore deploy
    ```

---

## Summary Checklist

- [x] Create Encore app and run locally
- [x] Build `monitor` and `site` services, set up APIs, and migrations
- [x] Add database migrations and run them
- [x] Implement ping/check logic and status APIs
- [x] Add Pub/Sub notification for uptime transitions
- [x] Integrate Slack notification and secret management
- [x] Deploy

---

**References:**
- [Encore Uptime Monitoring Tutorial](https://encore.dev/docs/ts/tutorials/uptime)
