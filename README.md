# eXo Platform Development with Che
[![Docker Stars](https://img.shields.io/docker/stars/exoplatform/exo-che.svg?maxAge=2592000)]() - [![Docker Pulls](https://img.shields.io/docker/pulls/exoplatform/exo-che.svg?maxAge=2592000)]()

## Overview

This repository contains all resources needed to develop a eXo Add-on with Che:
* an official **exoplatform/che** Docker image with:
 * Ubuntu
 * Oracle JDK
 * eXo Platform Community edition
 * LibreOffice
 * Maven
 * Git
* a folder for each eXo Addon with:
 * a **Dockerfile** to add the specific resources required by this Add-on
 * a **catalog.json** file used by Che custom commands
 * a **che-workspace.json** file to init a ready-to-work worskpace with this add-on

## Read the following documentation to know more about Che and eXo Platform

* [How to run Che Server with eXo Tasks locally](./doc#install-che)
* [How to add Che configuration for another eXo Add-on](./doc#contribute)
