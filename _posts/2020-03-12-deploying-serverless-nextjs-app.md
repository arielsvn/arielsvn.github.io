---
title: Deploying a serverless NextJS App
date: 2020-03-12 09:00:01 Z
categories:
  - javascript
  - nextjs
  - zeit
layout: post
summary: We migrated to NextJS recently and started using ZEIT to deploy our frontend. We required some configuration and even found a bug on ZEIT's platform in the process.
---

For a while, I have been wanting to try server-side rendering, specifically with [NextJS](https://nextjs.org/). Recently, a [big pull request](https://github.com/AlexsLemonade/refinebio-frontend/pull/853) landed in refinebio's repository with the entire migration.

We had a few discussions to decide how to deploy these changes. We liked [Labda@Edge](https://aws.amazon.com/lambda/edge/) and considered the idea of deploying it with [serverless](https://serverless.com/blog/serverless-nextjs/) since they had [a component](https://github.com/danielcondemarin/serverless-next.js/tree/master/packages/serverless-nextjs-component) specifically for NextJS.

We also came across [ZEIT](https://zeit.co), they seemed to offer the ability to deploy a NextJS site to CDNs, just like `Lambda@Edge`, with no configuration. I usually get worried when I hear that, because hitting a corner case that's not covered it's bound to happen. Still, [we decided](https://github.com/AlexsLemonade/refinebio-frontend/issues/857) to give it a try.

### Considerations with ZEIT

The initial integration was really easy, we used their [github integration](https://zeit.co/github) and deployed to a `*.now.sh` free domain. However, two issues arose when we tried to set up two domains for staging and production.

The first one was a bug on ZEIT's platform. In our repo we have two main branches, `master` and `dev` which we use for production and staging respectively. The default one is `dev`. It seemed that ZEIT had some problems when identifying the default branch in the repository and they were not letting me deploy the `master` branch.

[<img src="/assets/img/2020-03-12-zeit-bug.gif" alt="Zeit bug with branch config" style="max-height: 700px; margin-left: auto; margin-right: auto;">](/assets/img/2020-03-12-zeit-bug.gif)

I filed a support ticket and talked with the engineers to resolve this. The experience was really good, and they addressed it in less than 48 hours. This was critical because I was starting to consider the idea of using a different platform to deploy our project.

The second problem we faced was setting up our staging server and have it use our staging API. In the frontend, this is controlled with an environment variable where we set the URL of the API that should be used.

At the time of writing ZEIT, still didn't support having different environment variables for each deployment, although they say they are [actively working on it](https://spectrum.chat/zeit/general/different-secrets-for-production-and-staging-environments~51846996-a72f-4f9e-8b21-7e07df89555b). To go around this, I had to disable the automatic deployments with their Github's integration and trigger the deploys manually with continuous integration (in our case [CircleCI](https://circleci.com/)).

I saw a [similar config](https://spectrum.chat/zeit/now/multiple-environments-via-github-deployments~34124374-4365-4abd-a53f-0698315adeb5?m=MTU0ODM3Njc3NTM2NQ==) and ended up with:

```yaml
version: 2
jobs:
  deploy-prod:
    docker:
      - image: circleci/python:3-node
    steps:
      - checkout
      - run: yarn global add now
      - run:
          command: $(yarn global bin)/now  --confirm --prod --token $ZEIT_TOKEN --local-config now.json --scope ccdl
  deploy-staging:
    docker:
      - image: circleci/python:3-node
    steps:
      - checkout
      - run: yarn global add now
      - run:
          command: $(yarn global bin)/now --confirm --prod --token $ZEIT_TOKEN --local-config now.staging.json --scope ccdl
workflows:
  version: 2
  test-and-deploy:
    jobs:
      - deploy-staging:
          filters:
            branches:
              only: dev
      - deploy-prod:
          filters:
            branches:
              only: master
```

The main difference between the two is the config files, the one for staging `now.staging.json` has the following

```json
{
  "version": 2,
  "alias": ["staging.refine.bio"],
  "name": "refinebio-frontend",
  "build": {
    "env": {
      "REACT_APP_API_HOST": "https://api.staging.refine.bio"
    }
  },
  "github": {
    "autoAlias": false,
    "silent": true
  }
}
```

The options under `github` serve two purposes:

- `autoAlias` is to disable [automatic production domain updates](https://docs-git-fork-karaggeorge-projects-changes.zeit.now.sh/docs/v2/advanced/now-for-github/#disabling-production-domain-updates) so that only the domains in `alias` get updated.

- `silent` disables [all comments on Github](https://docs-git-fork-karaggeorge-projects-changes.zeit.now.sh/docs/v2/advanced/now-for-github/#disable-commenting-with-silent-mode), by default there were comments in our pull requests every time they were deployed. We prefer to just use the links on the checks offered by Github.

### Conclusion

I'd still recommend ZEIT, I had a very good experience with their technical support and even if their "zero-configuration" tools don't work for your use case, they still offer other lower-level tools that can be customized further. Like the [Now CLI](https://github.com/zeit/now) command.

Pay special attention if you have a complicated staging/production setup, because you might need to use the [Now CLI](https://github.com/zeit/now) with a custom CI configuration like the one I had to use.