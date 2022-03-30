# Learning NGINX App Protect

## Introduction
NGINX App Protect is the combination of industry leading F5's advanced WAF technology and high performance NGINX Plus data plane. Read more [here](https://www.nginx.com/products/nginx-app-protect/web-application-firewall)

The purpose of this repo is to provoide an easy setup for learning NGINX NAP.
<br/>

# Setup

0. Build NAP Docker image</br>
Follow the instruction in the official doc to build a local docker container app-protect image. https://docs.nginx.com/nginx-app-protect/admin-guide/install/#docker-deployment


1. Clone the repo
```
git clone https://github.com/mcheo-nginx/learning-nap.git
cd learning-nap
```

2. Step up the stacks
```
docker-compose -f docker-compose.yaml up -d
```

The stack consists of 3 containers:
- NGINX App Protect instance
- Backend app server (httpbin)
- Elasticsearch for NAP dashboard


3. Complete Elasticsearch setup</br>
Use browser to visit http://localhost:5601, once the page loads successfully which means startup has completed, execute the following steps:
```
sh import_kibana_dashboard.sh
```

# Testing
Inside policies folder, there are many short policy files to demonstrate different policy control.

```
#Update nginx.conf, modify app_protect_policy_file directive to point to correct policy file name
#Reload nginx and wait for 2-3 seconds for the policy to take effect
docker exec learning-nap_nginx_1 nginx -s reload
```
NGINX NAP policy is a declarative policy, you may combine multiple controls into a single policy json file.

Here are some tips on how to test the policy. You may also check the Kibana dashboard for the outcome.

http://localhost:8000 (protected with NAP) http://localhost:8001 (unprotected), you may compare the outcome of with/without NAP policy.

* bot_restriction.json
  <br/>curl command will fail as it is classified as untrusted-bot, you may toggle the policy action to "alarm" or "block" to test it out
  
* custom_blocking_page.json
  <br/>Use browser to visit http://localhost:8000/get?<script>, this will cause violation error and we will see a custom blocking page
  
* file_type_restriction.json
  <br/>Use browser to browse http://localhost:8000/file.exe (this file doesn't exist, but nonetheless it is a violation or http://localhost:8000/robots.txt
  
* host_header_violation.json
  <br/>curl command to visit http://localhost:8000/get, http://127.0.0.1:8000/get or curl -H "Host: myapi.com" http://localhost:8000/get
  
* http_method_restriction.json
  <br/>curl -X PUT http://localhost:8000/put 
  
* parameters_restriction.json
  <br>curl http://localhost:8000/get?mandatory=abc&repeated=abc&empty=abc   Try with combination of either empty value, repeated param or missing mandatory
  
* url_restriction.json
  <br>curl http://localhost:8000/anything
  
* xff_ip_blacklist.json
  <br>curl -H "xff: 10.10.1.10" http://localhost:8000/get
  
* api_oas_reference.json
  <br>curl http://localhost:8000/delay/adfdf   this will cause illegal param type
  
 
# Dashboard
Visit http://localhost:5601 and click into Discover or Dashboard to see the traffics/violations.

Please refer here for extra [reference](https://devcentral.f5.com/s/articles/Dashboards-for-NGINX-App-Protect)
<br/>

# Reference
1. Please refer to [official guide](https://docs.nginx.com/nginx-app-protect/configuration-guide/configuration/) for more instruction on how to tune the policy
2. https://waffler.dev/prod/ is an useful tool to help construct the NAP policy
