# MDCB setup guide

### 0. Prequisites
Note -> These instructions are run on Linux. (AWS Linux 2)

Run these to install Docker, Docker-Compose & Git
```
sudo yum update -y
sudo yum install git -y
sudo yum install -y docker
sudo service docker start
sudo usermod -aG docker ec2-user ## If needed, on EC2
sudo su
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker ps
```

### 1. Setting up prerequisites for Tyk pro and Tyk MDCB

A) Clone repo
```
git clone https://github.com/TykTechnologies/tyk-pro-mdcb-docker-demo
cd tyk-pro-mdcb-docker-demo/mdcb
```

B) In the `mdcb` directory, create file named ".env".  We will load our env variables here to be used by docker-compose. The contents of the file:
```
DASHBOARD_LICENCE={add-your-dashboard-license-here}
MDCB_LICENCE={add-your-mdcb-license-here} 
```

### 2. Run Stack
`docker-compose up -d`

### 3. Bootstrap the install
Log on to the Dashboard via `http://<your-host>:3000`

### 4. Enable Hybrid on Master DC

Export the defaults
```
export DASH_ADMIN_SECRET=12345
export DASH_URL=localhost:3000
```

Export your Dashboard's ORG ID
You can find your organisation id in the Dashboard, under your user account details.
```
export ORG_ID=<YOUR_ORG_ID>
```

Download Org object
```
curl $DASH_URL/admin/organisations/$ORG_ID -H "Admin-Auth: $DASH_ADMIN_SECRET" | python -mjson.tool > myorg.json
```

Edit myorg.json to add this bit (replace existing ones")
```
"hybrid_enabled": true,
"event_options": {
    "key_event": {
        "redis": true
    },
    "hashed_key_event": {
        "redis": true
    }
},
```

Update org with new settings
```
curl -X PUT $DASH_URL/admin/organisations/$ORG_ID -H "Admin-Auth: $DASH_ADMIN_SECRET" -d @myorg.json
```

Response:
```
{"Status":"OK","Message":"Org updated","Meta":null}
```

### 5. Run tyk slave gw

This can be local DC or remote DC.  If remote DC, clone the repo again on the new DC.

A) go to worker folder
```
cd ../worker
```

B) Add these to tyk_worker.conf
- RPC key (org ID) to `slave_options.rpc_key`
- API key (user API key) `slave_options.api_key`
- connection_string without protocol (of VM) plus port 9090 to `slave_options.connection_string`, ie `10.45.166.51:9090` or `host.docker.internal:9090`
- Add a unique `group_id` to describe this worker cluster.  These must be unique for each Location/Data Center

C) run
```
docker-compose up -d
```
