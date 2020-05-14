## Euraknos deployment

DSpace installation instructions:

* [On GitHub](https://github.com/DSpace/DSpace/blob/master/dspace/src/main/docker-compose/README.md) (the one used)
* [On DSpace documentation website](https://wiki.lyrasis.org/display/DSPACE/Try+out+DSpace+7#TryoutDSpace7-InstallviaDocker)

### Deploy

Pull images:

```bash
docker-compose -f docker-compose.yml -f docker-compose-cli.yml pull
```

Build images:

```bash
docker-compose -f docker-compose.yml -f docker-compose-cli.yml build
```

Start DSpace API, dspace-angular, Virtuoso triplestore, SQL database and solr search index:

```bash
docker-compose -p d7 -f docker-compose.yml -f dspace/src/main/docker-compose/docker-compose-angular.yml up -d
```

> Access:
>
> * Angular UI at http://localhost:3000
> * DSpace API at http://localhost:8080/server
> * SOLR index at http://localhost:8983/solr/#/

### Load test data

Create admin account:

```bash
docker-compose -p d7 -f docker-compose-cli.yml run --rm dspace-cli create-administrator -e test@test.edu -f admin -l user -p admin -c en
```

> Login with `test@test.edu` and password `admin`

Ingest data:

```bash
docker-compose -p d7 -f docker-compose-cli.yml -f dspace/src/main/docker-compose/cli.ingest.yml run --rm dspace-cli
```

> See the [README for more details on test data](https://github.com/DSpace/DSpace/blob/master/dspace/src/main/docker-compose/README.md).

### Shutdown

Stop DSpace:

```bash
docker-compose -p d7 -f docker-compose.yml -f dspace/src/main/docker-compose/docker-compose-angular.yml down
```

Remove all existing Docker volumes:

```bash
docker volume rm $(docker volume ls -q)
```

---

## Enable RDF module

A docker-compose-rdf.yml has been added to deploy the Virtuoso triplestore.

### Deploy with RDF module

Enable RDF in the config files by uncommenting `rdf.enabled = true`

* dspace/config/dspace.cfg
* dspace/config/modules/rdf.cfg
* dspace/config/modules/rdf/constant-data-general.ttl

> Figure out the best way to provide RDF config

Pull images:

```bash
docker-compose -f docker-compose.yml -f docker-compose-cli.yml -f docker-compose-rdf.yml pull
```

Build images:

```bash
docker-compose -f docker-compose.yml -f docker-compose-cli.yml -f docker-compose-rdf.yml build
```

Start DSpace API, dspace-angular, Virtuoso triplestore, SQL database and solr search index:

```bash
docker-compose -p d7 -f docker-compose.yml -f docker-compose-rdf.yml -f dspace/src/main/docker-compose/docker-compose-angular.yml up -d
```

> Access at same URLs and:
>
> * Virtuoso SPARQL endpoint at http://localhost:8890/sparql

Shutdown DSpace with RDF module:

```bash
docker-compose -p d7 -f docker-compose.yml -f docker-compose-rdf.yml -f dspace/src/main/docker-compose/docker-compose-angular.yml down
```

### Notes

Enabling RDF has been tried at multiple places

* rdf.cfg
* dspace.cfg
* as env variable in docker-compose-rdf.yml

The following error is raised by Angular:

```bash
$ node dist/server.js dist/server.js
internal/modules/cjs/loader.js:960
  throw err;
  ^

Error: Cannot find module '/app/dist/server.js'
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:957:15)
    at Function.Module._load (internal/modules/cjs/loader.js:840:27)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:74:12)
    at internal/main/run_main_module.js:18:47 {
  code: 'MODULE_NOT_FOUND',
  requireStack: []
}
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
[nodemon] app crashed - waiting for file changes before starting...

webpack is watching the files…
```

The RDF module is clearly not active on the API (http://localhost:8080/rdf). I note if it is due to Virtuoso, our options are:

* Make sure the config we are provided are really taken into account by the DSpace API (hard to do during the build, and the `docker-compose up` do not print the enabled Modules)
* Trying on a more stable release (the latest master branch could be bugged)
* Replacing Virtuoso by Fuseki
  * Honestly Fuseki is more complex to deploy, it requires to pass a ttl file which create the empty dataset to use. Virtuoso just comes with a SPARQL endpoint ready.

### Additional RDF notes

* See the detailed config file for RDF
  * https://github.com/DSpace/DSpace/blob/master/dspace/config/modules/rdf.cfg
* This might need to be changed (the file seems to be used)
  *  https://github.com/DSpace/DSpace/blob/master/dspace/config/modules/rdf/constant-data-general.ttl#L5
* Fuseki deployment example with docker-compose:
  * https://github.com/DSpace-Labs/DSpace-Docker-Images/blob/master/docker-compose-files/dspace-compose/rdf.override.yml