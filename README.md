# hello-oodt

## Summary
This tutorial provides a simple introduction to using OODT services as Docker containers. 
It sets up a simple system composed of one OODT File Manager (FM) and one OODT Workflow Manager (WM),
running in separate Docker containers on the same host. The WM is configured to execute a "test-workflow" composed of 2 jobs ("PGEs"), each of which produces an output file. The FM is configured to archive the output files as products of type "TestFile". 

This tutorial is based on Docker images built from OODT 1.0.

## Pre-Requisites
* Docker engine installed on the host (Linux, Mac or Windows)
* Git

## Quick Start

1. Download the source code from this GitHub repository:

    git clone https://github.com/oodt-cloud/hello-oodt.git
    
    cd hello-oodt
  
2. Optionally: pre-download the OODT images (will make the next step faster):

    docker pull oodthub/oodt-filemgr:latest
    
    docker pull oodthub/oodt-wmgr:latest
    
3. Start the docker containers:

    docker-compose up -d
    
    docker-compose logs -f
    
 4. Submit a test-workflow (from inside the WM container):
 
    docker exec -it wmgr /bin/bash
    
    [root@abc123] cd $OODT_HOME/cas-workflow/bin
    
    [root@abc123] ./wmgr-client --url http://localhost:9001 --operation --sendEvent --eventName test-workflow --metaData --key Dataset abc --key Project 123  --key Run 1
    
    Follow the workflow execution in the WM log file: 
    [root@abc123] tail -f $OODT_HOME/cas-workflow/logs/cas_workflow.log
    
 5. Verify that products were generated and ingested (from inside the FM container):
 
    docker exec -it filemgr /bin/bash
    
    ls -l $OODT_HOME/jobs
    
    ls -l $OODT_ARCHIVE
    
    curl 'http://localhost:8983/solr/#/oodt-fm/query'
    


## Notes

* Configuration: the following directories contain the OODT configuration for this particular tutorial, and are cross-mounted from the local host into the Docker containers:

   ./

* Within each container, OODT services are started through supervisord. To start/stop/restart a service (within the appropriate container):

   supervisorctl restart oodt_wmgr
   
   supervisorctl restart oodt_filemgr
   
   configuration files: /etc/supervisor/supervisord.conf and /etc/supervisor/conf.d/\*.conf
   log file: /tmp/supervisord.log
   
* Log files

* URLs



* How to extend:
