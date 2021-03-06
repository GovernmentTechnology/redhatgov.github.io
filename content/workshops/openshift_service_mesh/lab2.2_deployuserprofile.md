---
title: Adding a New Service
workshops: openshift_service_mesh
workshop_weight: 22
layout: lab
---

# Adding a New Service to the Mesh

You built the user profile service and now you need to deploy it into the service mesh.

## Deploy Application

<blockquote>
<i class="fa fa-terminal"></i>
Navigate to the workshop directory:
</blockquote>

```
cd $HOME/openshift-microservices/deployment/workshop
```

The deployment file 'userprofile-deploy-all.yaml' was created for you to deploy the application.  The file creates the user profile service and an accompanying PostgreSQL database.  Similar to the other source files, an annotation 'sidecar.istio.io/inject' was added to tell Istio to inject a sidecar proxy and add this to the mesh.

<blockquote>
<i class="fa fa-terminal"></i>
Verify the annotation in the 'userprofile' file:
</blockquote>

```
cat ./openshift-configuration/userprofile-deploy-all.yaml | grep -B 1 sidecar.istio.io/inject
```

Output:
```
	annotations:
	  sidecar.istio.io/inject: "true"
  --
    annotations:
      sidecar.istio.io/inject: "true"
```

The annotation appears twice for the userprofile and PostgreSQL services.

Before deploying the service, you need a reference to the local image you built in the previous lab.

<blockquote>
<i class="fa fa-terminal"></i>
Run the following commands:
</blockquote>

```
USER_PROFILE_IMAGE_URI=$(oc get is userprofile -o jsonpath='{.status.dockerImageRepository}{"\n"}')
echo $USER_PROFILE_IMAGE_URI
```

Output (sample):
```
image-registry.openshift-image-registry.svc:5000/microservices-demo/userprofile
```

<blockquote>
<i class="fa fa-terminal"></i>
Deploy the service using this image URI:
</blockquote>

```
sed "s|%USER_PROFILE_IMAGE_URI%|$USER_PROFILE_IMAGE_URI|" ./openshift-configuration/userprofile-deploy-all.yaml | oc create -f -
```

<blockquote>
<i class="fa fa-terminal"></i>
Watch the deployment of the user profile:
</blockquote>

```
oc get pods -l deploymentconfig=userprofile --watch
```

> The userprofile service may error and restart if the PostgreSQL pod is not running yet.

Output:
```
userprofile-xxxxxxxxxx-xxxxx              2/2     Running		    0          2m55s
```

Similar to the other microservices, the user profile services run the application and the Istio proxy.

<blockquote>
<i class="fa fa-terminal"></i>
Print the containers in the 'userprofile' pod:
</blockquote>


```
oc get pods -l deploymentconfig=userprofile -o jsonpath='{.items[*].spec.containers[*].name}{"\n"}'
```

Output:
```
userprofile istio-proxy
```

## Access Application

The user profile service is deployed!  Let's test this in the browser.

<blockquote>
<i class="fa fa-desktop"></i>
Navigate to the 'Profile' section in the header.
</blockquote>

<p><i class="fa fa-info-circle"></i> If you lost the URL, you can retrieve it via:</p>
`echo $GATEWAY_URL`

<br>

You should see the following:

<img src="../images/app-profilepage.png" width="1024"><br/>
 *Profile Page*


Congratulations, you deployed the user profile service and added it to the service mesh!

{{< importPartial "footer/footer.html" >}}