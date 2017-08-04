# hello-oodt

## Summary
This tutorial provides a simple introduction to using OODT with Docker. 
It sets up a simple system composed of one OODT File Manager (FM) and one OODT Workflow Manager (WM),
running in separate Docker containers on the same host. The WM is configured to execute a "test-workflow" composed of 2 jobs ("PGEs"), each of which produces an output file. The FM is configured to archive the output files as products of type "TestFile".

## Pre-Requisites
* Docker engine installed on the host (Linux, Mac or Windows)
* Git
