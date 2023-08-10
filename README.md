# Requirements
- Node
- Netlify account
- GitLab account
- Availible GitLab runner (*NOTE: if you want register self-hosted runner check out my other repo [here](https://github.com/fayvori/iac-gitlab-runner)*)
- netlify-cli

# Implementation
First we need to create a new react app using `create-react-app` utility

![create react app](./docs/images/create-react-app.jpeg)

### **Also don't forget to create a new personal access token on netlify website [here](https://app.netlify.com/user/applications)**

After that create a new file inside app folder called `netlify.toml`. With following content:

```toml
[build]
  publish = "build"
```

The publish parameter represents our output folder after executing `npm run build` command.

Then we need to register our site on netlify site. I will process with command line. For register site on netlify run the following command inside app folder:

```bash
netlify sites:create -n YOU_DESIRED_SITE_DOMAIN (e.g. gitlabcicd-site-deploy)
```

Output should be like this:

![netlify sites:create output](./docs/images/netlify-create.jpeg)

And for now we are done with netlify part. Let's jump out into GitLab part!

Create a new gitlab repository as shown down below:

![gitlab repository creating process](./docs/images/create-gitlab-project.jpeg)

After that go to **Settings > CI/CD > Variables** and click *Add variable*. We need two variables in order to deploy our website to netlify `$NETLIFY_SITE_ID` `$NETLIFY_AUTH_TOKEN`.

![gitlab vars section](./docs/images/gitlab-vars.jpeg)

The next step is to set up out gitlab ci/cd pipeline. First create Dockerfile as shown below:

```Dockerfile
FROM node:18-alpine3.17 as build
WORKDIR /app

ENV PATH /app/node_modules/.bin:$PATH
COPY package.json /app/package.json
RUN npm install --silent
RUN npm install react-scripts -g --silent
COPY . /app
RUN npm run build

# production environment
FROM nginx:1.16.0-alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

And now we can finally proceed to our `.gitlab-ci.yml` file

```yaml
default:
  image: node:18-alpine3.17
  interruptible: false

variables:
  DOCKER_IMAGE: fayvori/react-gitlabcicd

stages:
  - test 
  - deploy 

cache:
  paths:
    - node_modules/

test:
  stage: test
  before_script:
    - npm i --silent
  script:
    - CI=true npm test
  tags:
    - netlify 
    - shared

deploy:
  stage: deploy
  only:
    - main
  before_script:
    - npm i --silent
    - npm run build
  script:
    - npx netlify-cli deploy --site $NETLIFY_SITE_ID --auth $NETLIFY_AUTH_TOKEN --dir build 
  tags:
    - netlify
    - shared
  when: manual
```

After push a new pipeline execution will be triggered (**NOTE:** in order to `deploy` appear you should push to main branch). 

Output of test job should be like this:

![test job output](./docs/images/tests.jpeg)

If test job was succeed you should be able to press `Play` button on `deploy` job

![manual deploy button](./docs/images/deploy-manual.jpeg)

Output of deploy job should contain `Website draft URL`
![output of deploy job](./docs/images/deploy-succeed.jpeg)

And after you click on that link our website will appear in browser

![website](./docs/images/react-app.jpeg)

**Thats it! Thanks for reading**
