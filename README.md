# hello-oodt

## Summary
This tutorial provides a simple introduction to using OODT services as Docker containers. 
It sets up a simple system composed of one OODT File Manager (FM) and one OODT Workflow Manager (WM),
running in separate Docker containers on the same host. The WM is configured to execute a "test-workflow" composed of 2 jobs ("PGEs"), each of which produces an output file. The FM is configured to archive the output files as products of type "TestFile". 

This tutorial is based on Docker images built from OODT 1.0. The system architecture is shown in the attached image file.

## Pre-Requisites
* Docker (for Linux, Mac or Windows)
* Git

## Quick Start

1. Download the source code from this GitHub repository:

    git clone https://github.com/oodt-cloud/hello-oodt.git
    
    cd hello-oodt
  
2. Optionally: pre-download the OODT images (will make the next step faster):

    docker pull oodthub/oodt-filemgr:latest
    
    docker pull oodthub/oodt-wmgr:latest
    
3. Start the docker containers:

    Open a terminal window, then:

    docker-compose up -d
    
    docker-compose logs -f
    
 4. Submit a test-workflow (from inside the WM container):
 
    Open another terminal window, then:
    
    docker exec -it wmgr /bin/bash
    
    [root@abc123] cd $OODT_HOME/cas-workflow/bin
    
    [root@abc123] ./wmgr-client --url http://localhost:9001 --operation --sendEvent --eventName test-workflow --metaData --key Dataset abc --key Project 123  --key Run 1
    
    Follow the workflow execution in the WM log file: 
    
    [root@abc123] tail -f $OODT_HOME/cas-workflow/logs/cas_workflow.log
    
    When the workflow has finished executing, the log file will show the message "Ingests were successful".
    
 5. Verify that products were generated and ingested (from inside the FM container):
 
    Open yet another terminal window, then:
 
    docker exec -it filemgr /bin/bash
    
    Inspect the job execution directory:
    
    [root@cde456] ls -l $OODT_HOME/jobs/\*
    
    List the content of the File Manager archive:
    
    [root@cde456] ls -l $OODT_ARCHIVE/test-workflow
    
    Make a metadata request to the Solr back-end:
    
    [root@cde456] curl 'http://localhost:8983/solr/oodt-fm/select?q=*%3A*&wt=json&indent=true'
    
    Inspect the File Manager log file:
    
    [root@cde456] cat $OODT_HOME/cas-filemgr/logs/cas_filemgr.log
    
6. Stop the running containers. In the first terminal window, type "ctrl-C" to end the Docker log display, then:

   docker-compose down

## Notes

### Important directories

The following directories contain the OODT configuration for this particular tutorial, and are cross-mounted from the local host into the Docker containers (WM or FM):
* WM and FM configurations: ./config:/usr/local/oodt/workflows
* PGEs (programs to be executed): ./pges:/usr/local/oodt/pges/

The following directories are managed by Docker but shared between the WM and FM containers:
* jobs: /usr/local/oodt/jobs
* archive: /usr/local/oodt/archive

### Access URLs

The OODT services can be accesed at the following URLs, from within the containers:

* Workflow Manager: http://wmgr:9001 (POST only)
* File manager: http://filemgr:9000 (POST only)
* Solr: 'http://filemgr:8983/solr/oodt-fm/select?q=*%3A*&wt=json&indent=true' (query for all products)

From outside the containers, the same URLs can be accessed as "localhost", or at the IP address used by the Docker Host engine.

### Start/Stop/Restart services

Within each container, OODT services are started through supervisord, but then deamonize themselves. The best way to stop/start/restart the services is to do it outside the containers using the docker-compose utilities, for example:

docker-compose stop/start/restart oodt_filemgr

docker-compose stop/start/restart oodt_wmgr

docker-compose stop/start/restart
   
### Log files

* Workflow Manager: $OODT_HOME/cas-workflow/logs/cas-workflow.log
* File Manager: $OODT_HOME/cas-filemgr/logs/cas_filemgr.log
* Supervisor: /tmp/supervisord.log

### How to extend

The WM and FM containers are configured to automatically look for workflows and product types in all sub-directories of the root directory $OODT_CONFIG = $OODT_HOME/workflows, which is mapped to the local host directory ./config. To add a new workflow:
* add the workflow and product type definitions to ./config/new-workflow/policy
* add the PGE configurations to ./config/new-workflow/pge-configs (referencing PGEs located under $PGE_ROOT)
* add the PGEs to ./pges/new-workflow

