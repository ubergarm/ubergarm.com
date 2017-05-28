Want to develop, build, test, and deploy consistent code across multiple platforms and even architectures?  Maybe work on your [Raspberry Pi](http://www.raspberrypi.org/) or [ODROID](http://www.hardkernel.com/) code from your laptop, push a Docker image, then pull the *exact* same code onto your target hardware device?  Maybe even automate all these steps using Continious Integration?!  Yeah why not! :)

## Background

I first tried this on [Circle CI](https://circleci.com/) but ran into some hiccups with with binfmt_misc not being quite full acessible on their build machines. While I like their user interface and ability to coordinate with others on github repos, binfmt_misc is a *must have* feature. When I asked about it, I received this curtious and efficient official response (Dec 2, 2015):

> 
Hello,
I dont think this is supported and it may not be for quite some time. 
Best,
Lev

So I tried [Travis CI](https://travis-ci.org/) and was pleasantly surprised to see everything worked straight away!

## Travis CI

Setup your Travis CI account to point to your desired github repo.  Place a `.travis.yml` in the top level directory of your repo, push, and watch the magic!

Here is a quick and dirty `.travis.yml` example from which you can setup your own requirements.

    sudo: required
    services:
      - docker
    # language: haskell
    # setup build machine
    before_install:
      - travis_retry sudo apt-get update
      - travis_retry sudo apt-get install -y --no-install-recommends qemu-user-static binfmt-support
      - sudo update-binfmts --display qemu-arm
    # build code
    install:
      - docker pull ubergarm/armhf-ubuntu:trusty
    # test code
    script:
      - uname -a
      - docker run --rm -it ubergarm/armhf-ubuntu:trusty uname -a
    # save/deploy build artifacts
    after_sucess:
      - /bin/true
    #  - docker login -e=$DOCKER_EMAIL -u=$DOCKER_USERNAME -p=$DOCKER_PASSWORD
    #  - docker push ubergarm/rowboat1

    # configure which branches are built
    branches:
      only:
        - builds
    #    - master

## Results

![results](/content/images/2015/12/travisci-qemu-docker-armhf.jpg)

## Conclusion

Continuous Integration comes to the embedded world!
