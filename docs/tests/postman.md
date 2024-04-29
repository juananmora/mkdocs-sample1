# How to create Postman API acceptance tests

Writing API acceptance tests is a great way to write system tests while hitting real environments. These tests can be run on your local machine as well as in Vapor Cloud (please consider what environments you're using and why). If it's your first time looking into testing with Postman, then consider reading through some of these resources:

- [Writing tests in Postman](http://blog.getpostman.com/2017/10/25/writing-tests-in-postman/)
- [API testing tips from a Postman professional](http://blog.getpostman.com/2017/07/28/api-testing-tips-from-a-postman-professional/)
- [JSON Schema](https://spacetelescope.github.io/understanding-json-schema/about.html)
- [Tiny Validator (for v4 JSON Schema)](https://geraintluff.github.io/tv4/)

## Creating a test collection

To keep things separate, the tests for your API collection should be created in a different collection. If your API collection is named `my-project` then the corresponding test collection should be named `my-project-tests`. Remember to share this collection the same way you would share a normal API collection.

Inside your test collection you will have endpoints for the endpoints you want to test. The order in which you arrange your folders and endpoints (from top to bottom) is **important** since this will be the order in which the Postman Runner will run your tests. Postman Runner is a tool for running all of your tests in a collection.

As with your normal API collection, top level folders should follow the domains of your project:

- Posts
- Authors
- Categories

Depending on the complexity of your project, you might want to create folders inside of these top level folders to describe flows.

Expanding the Posts folder might then reveal the following endpoints in an arranged order:

- Get all posts
- Add post
- Get single post
- Get all posts
- Delete post
- Get single post

## Getting Started

1. Install `newman`
2. Install `newman-reporter-influxdb`
3. Install InfluxDB (Get the server address, port, database name, etc)

### Prerequisites

1. `node` and `npm`
2. `newman` - `npm install -g newman`
3. [InfluxDB](https://github.com/influxdata/influxdb)

---

## Installation

```console
npm install -g newman-reporter-influxdb
```

> Installation should be done globally if newman is installed globally, otherwise install without `-g` option

---

## Usage

Specify `-r influxdb` option while running the collection

```bash
newman run <collection-url> -r influxdb \
  --reporter-influxdb-server <server-ip> \
  --reporter-influxdb-port <server-port> \
  --reporter-influxdb-name <database-name> \
  --reporter-influxdb-measurement <measurement-name>
```

- By default, reporter consider influxdb version 1.x (i.e 1.7, 1.8)
- In case of InfluxDB version 2, specify version, org and bucket name as well
  - `--reporter-influxdb-version 2`
  - `--reporter-influxdb-org <org-name>`
  - `--reporter-influxdb-name <bucket-name>`

Example:

```