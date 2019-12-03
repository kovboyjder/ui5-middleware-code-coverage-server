# ui5-middleware-code-coverage
A code coverage instrumenter for UI5-tooling

Middleware for [ui5-server](https://github.com/SAP/ui5-server), enabling code coverage.

This is a wrapper for the [Istanbul-Middleware](https://www.npmjs.com/package/istanbul-middleware). 

## Install

```bash
npm install ui5-middleware--code-coverage-server --save-dev
```

## Configuration options (in `$yourapp/ui5.yaml`)

- auth: `string` 
  The username and password seperated by a ':'
- client: `string`
  The client which you want to download from
- hostname : `string`
  The hostname of your Fiori server
- regExp: `Regular Expression
  Regular expression for you to pattern match which JS files should be covered. (Example: /(Agilux).*(.js)$/g) will test on the work Agilux is present and the file ends on .js
- enabled: `Boolean`
  switch to turn on or off coverage
## Usage

1. Define the dependency in `$yourapp/package.json`:

```json
"devDependencies": {
    "ui5-middleware-code-coverage-server": "*"
},
"ui5": {
  "dependencies": [
    "ui5-middleware-code-coverage-server"
  ]
}
```

> As the devDependencies are not recognized by the UI5 tooling, they need to be listed in the `ui5 > dependencies` array. In addition, once using the `ui5 > dependencies` array you need to list all UI5 tooling relevant dependencies.

2. configure it in `$yourapp/ui5.yaml`:

```yaml
server:
  customMiddleware:
  - name: ui5-middleware-code-coverage
    afterMiddleware: compression
    mountPath: /
    configuration:
      auth: username:password
      hostname: http(s)://{hostname}/{port}
      regExt: /(Agilux).*(.js)$/g)
      enabled: "false"


```


## How it works
The middleware will launch another express nodejs server that will act as a collector and instrumenter for your code. The path in the yaml configuration tells the package which folder and subsequent JS files needs to be instrumented for code coverage. 

The coverage is stored in your browser session in the `window.__coverage__` object. You can use this plugin together with [Selenium](https://medium.com/@the1mills/front-end-javascript-test-coverage-with-istanbul-selenium-4b2be44e3e98) and after execution of your scripts submit the code coverage or you can simply use it manually and after your testing submit the coverage to the server using below snippet.

The coverage server downloads the files that matches the regular expression to your local filesystem in a test_coverage folder. Then instruments the files for code coverage

```
$.ajax({
  type: "POST",
  url: 'http://localhost:3000/coverage/client',
  data: JSON.stringify(window.__coverage__),
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json'
}
});
```

## The Coverage service
The above snippet adds the following endpoints to your app under `/coverage`

<table>
<thead>
    <tr>
        <th>URL</th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td><code>GET&nbsp;/</code></td>
        <td>
            Dynamic code coverage HTML report showing live coverage.
            Clickable  with drill-downs just like the static version
        </td>
    </tr>
    <tr>
        <td><code>POST&nbsp;/reset</code></td>
        <td>Reset coverage to baseline numbers</td>
    </tr>
    <tr>
        <td><code>GET&nbsp;/download</code></td>
        <td>Download a zip file with coverage JSON, HTML and lcov reports</td>
    </tr>
    <tr>
        <td><code>POST&nbsp;/client</code></td>
        <td>
            Allows you to post a coverage object for client-side code coverage from the browser.
            Must be a JSON object with a <code>Content-type: application/json</code> header.
            This object is aggregated with the stats already present on the server
        </td>
    </tr>
</tbody>
</table>
