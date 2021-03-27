# How to use the template

This content shows how to install and deploy the site.

## Sections naming
A new development must correspond to a new feature branch name as _feature/section/topic-argument_. Where:
- _section_: could be basic, advanced, start
- _topic_: identifies the area into the section and corresponds to a new md file
- _argument_: a specific paragraph internal to a section (could be empty in case of starting of a new topic)

## Installation
To install the theme do the following:

```shell script
git submodule add https://github.com/google/docsy.git themes/docsy
git submodule update --init --recursive

sudo npm install -D --save autoprefixer
sudo npm install -D --save postcss-cli
sudo npm install -D postcss
```

## Update the theme

To update the theme you have to update the submodules:

```shell script
git submodule update --init --recursive
```

## Deploy the site

To test run:
```shell script
hugo server -p 8080
```

To build the site type:
```shell script
hugo
```

it will create the public folder into the project. To execute in an `nginx container` type:

```shell script
docker run --rm --name some-nginx \
-v $PWD/public:/usr/share/nginx/html:ro \
-p 8080:80 -d nginx
```
for major info take a look [here](https://hub.docker.com/_/nginx).
