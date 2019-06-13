EAP S2I One-off Patch Example using a configmap and the embedded-cli:
===============

Note, this example illustrates a means to apply a one-off patch to EAP using a configmap as the placeholder for the patch zip archive.  Note that this approach may be limited by the size of the patch zip archive file.  If creating the configmap succeeds, then the rest of this process should work.


Basic instuctions: 

- Create a ConfigMap with patch:

  ```$ oc create -n myproject configmap jbeap-16108.zip --from-file=jbeap-16108.zip```

- Update extensions/patch.cli to install the required patch.
   - for multiple patches, seperate .cli files for each patch application could be created, and applied in any required order from extensions/postconfigure.sh

- The project should have a .s2i/environment with the following contents:
    
    ```CUSTOM_INSTALL_DIRECTORIES=extensions```
  
  (Alternatively, this may be provided to oc new-app with -e CUSTOM_INSTALL_DIRECTORIES, but easy to forget :) )

- The application template or existing deployment config may now be modified to mount the configmap containing the patch and the application rebuilt, see: https://github.com/travisrogers05/eap-patch-using-configmap-cli/blob/master/eap71-basic-s2i-patching.json#L344-#L351 and https://github.com/travisrogers05/eap-patch-using-configmap-cli/blob/master/eap71-basic-s2i-patching.json#L432-#L436 for the required mount configuration. 

- The volume name should match the configmap created in the initial ConfigMap creation (jbeap-16108.zip, in this example.) 

- install / replace the template: 
   ``` $ oc -n openshift replace --force -f eap71-basic-s2i-patching.json ```

- create / recreate the application:
     ``` $ oc new-app --template=eap71-basic-s2i-patching \
       -p SOURCE_REPOSITORY_URL="https://github.com/travisrogers05/eap-patch-using-configmap-cli" \
       -p SOURCE_REPOSITORY_REF="master" \
       -p CONTEXT_DIR="" \
       -p APPLICATION_NAME="eap-patching-demo" ```
- Alternativly, the deployment controller configurtion can be modified to add the required volumes. Modification of the deployment controller is similar to the template configuration changes (use ```oc edit dc/eap-app-name```, or edit the yaml in the OpenShift console).

- When the server boots, you should see something similar to:
    > 03:19:08,195 INFO  [org.jboss.as.patching] (MSC service thread 1-2) WFLYPAT0050: JBoss EAP cumulative patch ID is: jboss-eap-7.1.5.CP, one-off patches include: eap-715-jbeap-16108
