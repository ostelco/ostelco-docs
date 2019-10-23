# The current work-flow shown in "the big picture":

![current-big-picture.png](current-big-picture.png)

## Deploying Prime:
* The developer creates a feature branch on local machine
* He/she tests the particular feature on the local computer, using:
  * Gradle clean build
  * Docker-compose up ; which runs a special container, which has Acceptance tests in it. Compose uses secrets from the local disk, (pantel-prod.json), and some SSL certs for nginx. The docker compose system uses docker-compose.yml and docker-compose.override.yml
  * It is important to understand that this “ability to run everything locally” has big advantage for the developer, as he/she can do all sorts of code changes, without needing to touch (push to) remote repository. 
  * Managing secrets mentioned in (2.b) is one of the main problems to be solved.* The developer then creates a PR for the feature branch
* TravisCI, which is watching this repo, picks up changes, and runs:
  * Gradle clean build (mandatory)
  * Unit tests (mandatory)
  * Code coverage (optional - but runs at the moment)
    * Codacy
    * Codebeat
  * Marks the PR as merge-able or not-merge-able (if there are merge conflicts)
* After code-review the (feature) PR is merged manually to the Develop branch
* Google Cloud Builder, which is watching both Develop and Master branches, sees a change in Develop branch and builds docker images. It uses cloudbuild.yml file for this step, and next two steps.
* It (GCB) then stores these images in Google Container Registry (GCR)
* GCB then deploys the docker images in a DEV cluster.
* When all is good, the developer then creates a PR from Develop branch to Master branch. Since Develop branch already has the code which is already tested through Travis, there is no need to run the tests again. So a simple PR is created from Develop to Master.
* The developer creates a git tag on the master branch, which is picked by Google Cloud Builder.
  * Since this is a monorepo, the tag name has the app name as prefix. E.g. prime-X.Y.Z. This implies `prime` is to be built and deployed. Similar to prime, there is separate builder for `pseudonym server` which looks for matching tag name with different regex.
  * But, since we want direct deployment and not tag based, the "new" build script will deploy all apps.
* GCB builds the docker images and pushes them to GCR
* GCB then deploys the docker images on the production cluster.

## Deploying OCS Gateway:
* Run script ocsgw/infra/script/deploy_ocsgw.sh
