<h1 align="center">
  <br>
    🔍 gowitness
  <br>
  <br>
</h1>

<h4 align="center">A golang, web screenshot utility using Chrome Headless.</h4>
<p align="center">
  <a href="https://twitter.com/leonjza"><img src="https://img.shields.io/badge/twitter-%40leonjza-blue.svg" alt="@leonjza" height="18"></a>
  <a href="https://goreportcard.com/report/github.com/sensepost/gowitness"><img src="https://goreportcard.com/badge/github.com/sensepost/gowitness" alt="Go Report Card" height="18"></a>
  <a href="https://github.com/sensepost/gowitness/actions/workflows/docker.yml"><img alt="Docker build & Push" src="https://github.com/sensepost/gowitness/actions/workflows/docker.yml/badge.svg"></a>
</p>
<br>

## introduction

`gowitness` is a website screenshot utility written in Golang, that uses Chrome Headless to generate screenshots of web
interfaces using the command line, with a handy report viewer to process results. Both Linux and macOS is supported,
with Windows support mostly working.

Inspiration for `gowitness` comes from [Eyewitness](https://github.com/ChrisTruncer/EyeWitness). If you are looking for
something with lots of extra features, be sure to check it out along with
these [other](https://github.com/afxdub/http-screenshot-html) [projects](https://github.com/breenmachine/httpscreenshot)
.

## features

The primary of purpose that gowitness serves is to take screenshots of websites, and do that well. As such, gowitness
has many ways to be told how to take a screenshot. Those inlude:

- Taking a single screenshot with the `single` command
- Scanning a network CIDR (or many) with the `scan` command
- Parsing a Nmap file for target information with the `nmap` command
- Accepting URL's submitted via a web service with the `server` command (perfect for integration with other tools)
- Taking screenshots from a text file (or read via stdin) with the `file` command

- In addition, gowitness can be configured in many ways.

### db

By default, gowitness will store screenshots in a `screenshots/` directory in the path where gowitness is being run
from. It will also store all the preflight information (initial HTTP response which includes headers & TLS
information) in a sqlite3 database called `gowitness.sqlite3`. The behaviour can be modified with the `--db-path`,
`--screenshot-path` and `--disable-db` flags.

### full page screenshots

To have full page screenshots taken, use the `--fullpage` / `-F` flag. For example:

`gowitness --fullpage single https://news.ycombinator.com/`

## usage

Examples of how to run some commands in gowitness are available in the help menus. Add the --help flag for any sub
command for verbose help information. For example:

`gowitness single --help`

Takes a screenshot of a single given URL and saves it to a file.
If no --output is provided, a filename for the screenshot will
be automatically generated based on the given URL. If an absolute
output file path is given, the --destination parameter will be
ignored.

Usage:
gowitness single [URL] [flags]

Examples:

- `gowitness single https://twitter.com`
- `gowitness single --destination ~/tweeps_dir https://twitter.com`
- `gowitness --disable-db single --destination ~/tweeps_dir https://twitter.com`
- `gowitness single -o /screenshots/twitter.png https://twitter.com`
- `gowitness single --destination ~/screenshots -o twitter.png https://twitter.com`

### examples

- screenshot a single website `gowitness single https://www.google.com/`
- screenshot a cidr using 20 threads `gowitness scan --cidr 192.168.0.0/24 --threads 20`
- screenshot open http services from a namp file `gowitness nmap -f nmap.xml --open --service-contains http`
- run the report server `gowitness report serve`

## api

An API is available for programmatic access to the database and or to trigger screenshots. To access the API, the web
interface needs to be started. This can be done with gowitness server. Use the --help flag to learn how to set an
alternate listen port / address.

Note: gowitness was built with the idea that the reporting server will only be served on localhost, and as such has no
authentication mechanism built in. If you want to expose the API server to a wider network then it is highly encouraged
that you place authentication in front of the server. An example of how to do that using Traefik is available here.

### Endpoints

An up-to-date reference on the API endpoints available can be found in the source code. All endpoints live under
the `/api/` path. For example, `/api/list`, `/api/detail/:id`.

- `GET /list`
- `GET /detail/:id `
- `GET /detail/:id/screenshot`
- `POST /screenshot`

### get list

Get's all the screenshots in the database, returned as a JSON formatted array. The ID field in the payload should be
used as a reference in a following /detail/* call to fetch the appropriate details.

Example call:

`curl localhost:7171/api/list`

Example Response:

```json
[
  {
    "ID": 1,
    "URL": "https://whatsapp.com",
    "FinalURL": "https://www.whatsapp.com/",
    "ResponseCode": 200,
    "Title": "Whatsapp"
  },
  {
    "ID": 2,
    "URL": "https://bing.com",
    "FinalURL": "https://www.bing.com:443/?toWww=1&redig=DB01213BD54C426785DF89E5C3DAFFCF",
    "ResponseCode": 200,
    "Title": "Bing"
  }
]
```

### get detail

- HTTP API: `GET /detail/:id`
- Parameters: `:id` is the id of the URL to query

Get details about a screenshotted URL. This endpoint can be quite verbose as it includes the DOM, all console logs,
network logs, headers and TLS information that could be saved.

Example call:

`curl localhost:7171/api/detail/2`

Example Response:

```json
{
  "ID": 2,
  "CreatedAt": "2022-06-10T15:48:48.411458+02:00",
  "UpdatedAt": "2022-06-10T15:48:48.558231+02:00",
  "DeletedAt": null,
  "URL": "https://bing.com",
  "FinalURL": "https://www.bing.com:443/?toWww=1&redig=DB01213BD54C426785DF89E5C3DAFFCF",
  "ResponseCode": 200,
  "ResponseReason": "200 OK",
  "Proto": "HTTP/1.1",
  "ContentLength": -1,
  "Title": "Bing",
  "Filename": "https-bing.com.png",
  "IsPDF": false,
  "PerceptionHash": "p:9bcb1df47430308f",
  "DOM": "<html></html>",
  "TLS": {
    "ID": 2,
    "CreatedAt": "2022-06-10T15:48:48.412845+02:00",
    "UpdatedAt": "2022-06-10T15:48:48.412845+02:00",
    "DeletedAt": null,
    "URLID": 2,
    "Version": 771,
    "ServerName": "www.bing.com",
    "TLSCertificates": [
      {
        "ID": 3,
        "CreatedAt": "2022-06-10T15:48:48.413079+02:00",
        "UpdatedAt": "2022-06-10T15:48:48.413079+02:00",
        "DeletedAt": null,
        "TLSID": 2,
        "Raw": null,
        "DNSNames": [
          {
            "ID": 8,
            "CreatedAt": "2022-06-10T15:48:48.413468+02:00",
            "UpdatedAt": "2022-06-10T15:48:48.413468+02:00",
            "DeletedAt": null,
            "TLSCertificateID": 3,
            "Name": "www.bing.com"
          }
        ]
      }
    ]
  },
  "Headers": [
    {
      "ID": 9,
      "CreatedAt": "2022-06-10T15:48:48.414665+02:00",
      "UpdatedAt": "2022-06-10T15:48:48.414665+02:00",
      "DeletedAt": null,
      "URLID": 2,
      "Key": "P3p",
      "Value": "CP=\"NON UNI COM NAV STA LOC CURa DEVa PSAa PSDa OUR IND\""
    },
    {
      "ID": 10,
      "CreatedAt": "2022-06-10T15:48:48.414665+02:00",
      "UpdatedAt": "2022-06-10T15:48:48.414665+02:00",
      "DeletedAt": null,
      "URLID": 2,
      "Key": "Set-Cookie",
      "Value": "SUID=M;"
    }
  ],
  "Technologies": [
    {
      "ID": 20,
      "CreatedAt": "2022-06-10T15:49:30.410426+02:00",
      "UpdatedAt": "2022-06-10T15:49:30.410426+02:00",
      "DeletedAt": null,
      "URLID": 52,
      "Value": "Google Font API"
    },
    {
      "ID": 21,
      "CreatedAt": "2022-06-10T15:49:30.410426+02:00",
      "UpdatedAt": "2022-06-10T15:49:30.410426+02:00",
      "DeletedAt": null,
      "URLID": 52,
      "Value": "React"
    }
  ],
  "Console": [
    {
      "ID": 92,
      "CreatedAt": "2022-06-10T15:49:30.410989+02:00",
      "UpdatedAt": "2022-06-10T15:49:30.410989+02:00",
      "DeletedAt": null,
      "URLID": 52,
      "Time": "0001-01-01T00:00:00Z",
      "Type": "console.error",
      "Value": "\"[GSI_LOGGER]: Check credential status returns invalid response.\""
    }
  ],
  "Network": [
    {
      "ID": 4306,
      "CreatedAt": "2022-06-10T15:49:30.412363+02:00",
      "UpdatedAt": "2022-06-10T15:49:30.412363+02:00",
      "DeletedAt": null,
      "URLID": 52,
      "RequestID": "88745.136",
      "RequestType": 0,
      "StatusCode": 0,
      "URL": "https://url.com",
      "FinalURL": "",
      "IP": "",
      "Time": "2022-06-10T06:51:21.013651999+02:00",
      "Error": "net::ERR_CONNECTION_REFUSED"
    }
  ]
}
```

### get screenshot

- HTTP API: `GET /detail/:id/screenshot`
- Parameters: `:id` is the id of the URL to query

Get the raw screenshot taken. Depending on if IsPDF is true from the above /detail/:id endpoint, the response would be
either a PDF or a PNG file.

Example call:

`curl localhost:7171/api/detail/52/screenshot`

Example Response:

`Binary data to be saved to a file.`

### take a screenshot

- HTTP API: `POST /screenshot`
- Parameters: A post body containing the URL to be processed and a flag if the screenshot should be returned. Optionally
  you may specify additional headers to send along with the screenshot request. Post body example:

```json
{
  "url": "https://google.com",
  "oneshot": false,
  "headers": [
    "foo: bar",
    "baz: foo"
  ]
}
```

Take a screenshot of the `url` specified. If oneshot is set to true, the screenshot will be taken and reflected back to
the requester. If oneshot is set to false, the screenshot will be processed in the background, and the results persisted
to the database.

Example request:

request a screenshot to be reflected back without persisting it to the database

`curl -X POST http://localhost:7171/api/screenshot --data '{"url": "https://google.com", "oneshot": "true"}'`

request a screenshot to be processed in the background

`curl -X POST http://localhost:7171/api/screenshot --data '{"url": "https://google.com", "oneshot": "false"}'`

request a screenshot to be processed in the background, with extra headers

`curl -X POST http://localhost:7171/api/screenshot --data '{"url":"<url>", "oneshot": "false", "headers": ["foo: bar", "baz: foo"]}'`

Example response (will only return json if oneshot was false. otherwise a png image will be returned):

```json
{
  "status": "created"
}
```

## installation

Installing gowitness is really easy, and you have a few options to do so.

After installing Google Chrome for your Operating System:

- If you have your Golang binary path in your $PATH, use `go get -u github.com/sensepost/gowitness`.
  Use `go install github.com/sensepost/gowitness@latest` if you have Golang 1.17+
- Download prebuilt binaries from the releases page.
- Compiling from source.
- Via docker.

- Once installed, you should be able to use the gowitness command.

```sh
$ gowitness
A commandline web screenshot and information gathering tool by @leonjza

Usage:
gowitness [command]

Available Commands:
file        screenshot URLs sourced from a file or stdin
help        Help about any command
nmap        Screenshot services from an Nmap XML file
report      Work with gowitness reports
scan        Scan a CIDR range and take screenshots along the way
server      Starts a webservice that takes screenshots
single      Take a screenshot of a single URL
version     Prints the version of gowitness

Flags:
-D, --db-path string           destination for the gowitness database (default "gowitness.sqlite3")
--debug                    enable debug logging
--disable-db               disable all database operations
--disable-logging          disable all logging
-F, --fullpage                 take fullpage screenshots
-h, --help                     help for gowitness
-X, --resolution-x int         screenshot resolution x (default 1440)
-Y, --resolution-y int         screenshot resolution y (default 900)
-P, --screenshot-path string   store path for screenshots (use . for pwd) (default "screenshots")
--timeout int              preflight check timeout (default 10)
--user-agent string        user agent string to use (default "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36")

Use "gowitness [command] --help" for more information about a command.
```

### building from source

The Makefile contains a number of targets to help you build gowitness from source. The general flow to build is:

- Ensure that you have at least golang version 1.16.
- Ensure that you have the relevant toolchain installed if you want to cross-compile. (The `Makefile` uses docker
  containers to achieve this for release builds)
- Clone this repository and `cd` into it.
- Build binaries with the `make` command. You can specify a specific target for `make`, such as `make linux` too.

## docker

A docker image is available on Dockerhub and Github Container Registry. You can get it by running either:

`docker pull leonjza/gowitness`

or

`docker pull ghcr.io/sensepost/gowitness:latest`

Depending on the image you choose, you may need to replace the `leonjza/gowitness` in the steps below
with `ghcr.io/sensepost/gowitness:latest`.

### usage

Using the docker container means you need to take into account that the gowitness binary will run in the container, but
in order for you to access the database and files generated by gowitness, you need to mount a volume into the container
to persist those.

The basic way to invoke gowitness via docker, without saving anything is:

`docker run --rm leonjza/gowitness gowitness`

However, that itself is not very useful, as any output generated (db & screenshots) will get blown away when the command
finishes and the container is cleaned up. Instead, we can mount in a volume to persist data. Do that with:

`docker run --rm -v $(pwd):/data leonjza/gowitness gowitness`

This way, the current working directory will get the data (db & screenshots) gowitness generates.

### screenshots

A more complete example of taking a screenshot with docker is therefore:

`docker run --rm -v $(pwd):/data leonjza/gowitness gowitness single https://www.google.com`

This will create a gowitness.sqlite3 database and a screenshots/ directory in the current working directory just as if
the golang binary was invoked from your local system.

### report server

The gowitness report server by default listens on localhost on port 7171. In the docker world, this server needs to be
told to listen on all interfaces, and then a mapping needs to be added to expose that port to your host. This can be
done with:

`docker run --rm -v $(pwd):/data -p7171:7171 leonjza/gowitness gowitness report serve --address :7171`

This command should be run in the directory where the gowitness.sqlite3 and screenshots/ file and directory lives. Of
course, if you customised those in the screenshotting phase, you would need to update the paths accordingly.

### accessing files / scans in the container

If you have a nmap file or a targets list you would like to access in the container, using the volume mount to /data
means the gowitness binary in the container will find your files there. For example:

`docker run --rm -v $(pwd):/data -p7171:7171 leonjza/gowitness gowitness nmap -f /data/nmap.xml`

### docker-compose

An example `docker-compose.yml` file is provided [here](./docker-compose.yml) which is configured with Traefik to have
authentication enabled in front of the report server. This way you could expose the server to a wider network to
consume.

## documentation

For installation information and other documentation, please refer to the
wiki [here](https://github.com/sensepost/gowitness/wiki).

## screenshots

![dark](images/gowitness-detail.png)

## credits

`gowitness` would not have been posssible without these amazing projects:

- [chromedp](https://github.com/chromedp/chromedp)
- [tabler](https://github.com/tabler/tabler)
- [zerolog](https://github.com/rs/zerolog)
- [cobra](https://github.com/spf13/cobra)
- [gorm](https://github.com/go-gorm/gorm)
- [go-nmap](https://github.com/lair-framework/go-nmap)
- [wappalyzergo](https://github.com/projectdiscovery/wappalyzergo)
- [goimagehash](https://github.com/corona10/goimagehash)

And many more!

## license

`gowitness` is licensed under a [GNU General Public v3 License](https://www.gnu.org/licenses/gpl-3.0.en.html).
Permissions beyond the scope of this license may be available at <http://sensepost.com/contact/>.
