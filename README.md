
# detect-secrets-docker

This repository contains instructions to preparing your repository for being scanned for secrets.
Credits to Yelp for their awesome [detect-secrets](https://github.com/Yelp/detect-secrets) repository for building this.

It also provides details on how you can include it into a CI tool like Cloud Build, using docker images

## Add the baseline file to whitelist repository's current content

To ensure that files like `package-lock.json` where the hash values are "secret-like", do not get mistaken as a secret, baseline filescan be created to "whitelist" such content. Follow the following instructions to create the baseline file:

1. Install docker
2. Clone this repository
3. Run the following command at the root directory of this repository to build the required docker image for later steps

```
docker build -f Dockerfile.base -t detect-secrets-docker .
```

4. Run the following command at the directory of your targetted repository, for generating the baseline files

```
docker run --name detect-secrets-docker -v $(pwd):/opt --entrypoint "create-basefiles" detect-secrets-docker
```

5. Remove the container once you're done

```
docker rm -f detect-secrets-docker
```

6. Commit the new file `.secrets.baseline` and `.secrets.lasthash` for your target repository


## Update the baseline file

In the case of new files with secret-like values, we want to whitelist those files too. Commit those affected files first, then run the following:


## Running this in your own CI pipelines

If your CI uses docker images, you can build a custom image for your CI (amend the tag according to your image repository requirements):

```
docker build -f Dockerfile.ci -t detect-secrets-docker-ci:latest .
```

In your CI, run the docker image with your code base mounted to the `/opt` directory. If new secrets are found between the latest commit towards the last time the baseline files are committed, a non-zero code will be returned, and should cause your build pipeline to fail

## Checking it locally?

If you want to run it locally, run the docker image build command under [here](https://github.com/Weiyuan-Lane/detect-secrets-docker#running-this-in-your-own-ci), then run the following command at the root directory of the repository that you are validating:

```
docker run --name detect-secrets-docker -v $(pwd):/opt --entrypoint "update-basefiles" detect-secrets-docker
```

