---

title: Vault Standup
author: ytjohn
date: 2017-10-14 02:18:59

layout: post

slug: vault-standup

---
This is a little walkthrough of settng up a "production-like" vault server with etcd backend (Not really production, no TLS and one person with all the keys). [Hashicorp Vault](https://www.vaultproject.io/) is incredibly easy to setup. Going through the dev walkthrough is pretty easy, but when you want to get a little more advanced, you start getting bounced around the documentation. So these are my notes of setting up a vault server with an etcd backend and a few policies/tokens for access. Consider this part 1, and in "part 2", I'll setup an ldap backend.

Q: Why etcd instead of consul?  
A: Most of the places I know that run consul, run it across multiple datacenters, and a few thousand servers, and interacts with lots of different services. Even if the secrets are protected, the metadata is quite visible. I want a rather compact and isolated backend for my eventual cluster.

Let's get started.

First off, create a configuration file for vault.

vaultserver.hcl:
```
metaladmin@vaultcore01:~$ cat vaultserver.hcl
storage "etcd" {
  address  = "http://localhost:2379"
  etcd_api = "v2"
  path = "corevault"
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 1
}

disable_mlock = true
cluster_name = "corevault"
```

Start the server (in its own terminal)

```
metaladmin@vaultcore01:~$ vault server -config=vaultserver.hcl
==> Vault server configuration:

                     Cgo: disabled
              Listener 1: tcp (addr: "0.0.0.0:8200", cluster address: "0.0.0.0:8201", tls: "disabled")
```

Init the server

```
dfzmbp:~ ytjohn$ export VAULT_ADDR=http://vaultcore01.pool.lab.ytnoc.net:8200
dfzmbp:~ ytjohn$ vault init
Unseal Key 1: f9XJwuxla/H86t8pbWVPnI6Tfi3nQtkasq303Oi8B+ep
Unseal Key 2: jFqEmE1c/lei+C1aIju6JM2t5fSI534g26E7Nv83t9RV
Unseal Key 3: ty/P+Jubm1BukPcdZ16eJFD0JQ9BFGqOSgft35/fvHXr
Unseal Key 4: 6k4aPjuKgz0UNe+hTVAOKUzrIvbS9w8UszB0HX3Au496
Unseal Key 5: PYNjRe9vBvHAGE9peiotrtjoYuVlAV/9QJ0NvqZScd2a
Initial Root Token: b6eac78d-f278-4d32-6894-a8168d055340
```

That Initial Root Token is your only means of accessing the vault once it&#039;s unsealed. Don&#039;t lose it until you replace it.

And this creates a directory in etcd (or consul)

```
metaladmin@vaultcore01:~$ etcdctl ls
/test1
/corevault
metaladmin@vaultcore01:~$ etcdctl ls /corevault
/corevault/sys
/corevault/core
```

Unseal it:

```
dfzmbp:~ ytjohn$ vault unseal
Key (will be hidden):
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 1
Unseal Nonce: d860cb16-f084-925d-6f41-d80ef15e297c
dfzmbp:~ ytjohn$ vault unseal
Key (will be hidden):
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 2
Unseal Nonce: d860cb16-f084-925d-6f41-d80ef15e297c
dfzmbp:~ ytjohn$ vault unseal
Key (will be hidden):
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
Unseal Nonce:
dfzmbp:~ ytjohn$ vault unseal
Vault is already unsealed.
```

Now let&#039;s take that root token and save it in our home directory. Not safe, because it&#039;s the all-powerful root token, you shold create a user token for yourself. But that&#039;s later.

Save your token (or export it as VAULT_TOKEN), then write and read some secrets.

```
echo b6eac78d-f278-4d32-6894-a8168d055340 > ~/.vault-token
dfzmbp:~ ytjohn$ vault read secret/hello
Key             	Value
---             	-----
refresh_interval	768h0m0s
value           	world

dfzmbp:~ ytjohn$ vault read -format=json secret/hello
{
	"request_id": "a4b199e7-ff7c-e249-2944-17424bf1f05c",
	"lease_id": "",
	"lease_duration": 2764800,
	"renewable": false,
	"data": {
		"value": "world"
	},
	"warnings": null
}

dfzmbp:~ ytjohn$ helloworld=`vault read -field=value secret/hello`
dfzmbp:~ ytjohn$ echo $helloworld
world
```

Ok, that's the basics of getting vault up and running. Now we want to get more users to access it. What I want is to create three "users" and give them each a path.

infra admins = able to create, read, and write to `secret/infra/*`
infra compute = work within the `secret/infra/compute` area.
infra network = work within the `secret/infra/network` area

infraadmin.hcl
```
path &quot;secret/infra/*&quot; {
  capabilities = [&quot;create&quot;]
}

path &quot;auth/token/lookup-self&quot; {
  capabilities = [&quot;read&quot;]
}
```

infracompute.hcl
```
path &quot;secret/infra/compute/*&quot; {
  capabilities = [&quot;create&quot;]
}

path &quot;auth/token/lookup-self&quot; {
  capabilities = [&quot;read&quot;]
}
```

infranetwork.hcl
```
path &quot;secret/infra/network/*&quot; {
  capabilities = [&quot;create&quot;]
}

path &quot;secret/infra/compute/obm/*&quot; {
  capabilities = [&quot;read&quot;]
}

path &quot;auth/token/lookup-self&quot; {
  capabilities = [&quot;read&quot;]
}
```

Now, we write these policies in.

```
dfzmbp:vault ytjohn$ vault policy-write infraadmin infraadmin.hcl
Policy &#039;infraadmin&#039; written.
dfzmbp:vault ytjohn$ vault policy-write infracompute infracompute.hcl
Policy &#039;infracompute&#039; written.
dfzmbp:vault ytjohn$ vault policy-write infranetwork infranetwork.hcl
Policy &#039;infranetwork&#039; written.
```

Let's create a token "user" for each policy.

```
dfzmbp:vault ytjohn$ vault token-create -policy=&quot;infraadmin&quot;
Key            	Value
---            	-----
token          	d16dd3dc-cd9e-15e1-8e41-fef4168a429e
token_accessor 	50a1162f-58a2-474c-466d-ec68fac9a2f9
token_duration 	768h0m0s
token_renewable	true
token_policies 	[default infraadmin]

dfzmbp:vault ytjohn$ vault token-create -policy=&quot;infracompute&quot;
Key            	Value
---            	-----
token          	d156326d-1ee6-7a93-d9d3-428e2211962d
token_accessor 	daf3beb4-6c31-4115-2d00-ba811c50b05b
token_duration 	768h0m0s
token_renewable	true
token_policies 	[default infracompute]

dfzmbp:vault ytjohn$ vault token-create -policy=&quot;infranetwork&quot;
Key            	Value
---            	-----
token          	84faa448-20d9-b472-349f-1053c81ff4c9
token_accessor 	68eea7ec-78c0-4be1-03c4-f2ec155b66de
token_duration 	768h0m0s
token_renewable	true
token_policies 	[default infranetwork]
```

Let's login as with the infranetwork token and attempt to write to compute. I have not yet created `secret/infra/compute` or `secret/infra/network` and I'm curious if infraadmin is needed to make those first.

```
dfzmbp:vault ytjohn$ vault auth 84faa448-20d9-b472-349f-1053c81ff4c9
Successfully authenticated! You are now logged in.
token: 84faa448-20d9-b472-349f-1053c81ff4c9
token_duration: 2764764
token_policies: [default infranetwork]
dfzmbp:vault ytjohn$ vault write secret/infra/compute/notallowed try=wemust
Error writing data to secret/infra/compute/notallowed: Error making API request.

URL: PUT http://vaultcore01.pool.lab.ytnoc.net:8200/v1/secret/infra/compute/notallowed
Code: 403. Errors:

* permission denied
dfzmbp:vault ytjohn$ vault write secret/infra/network/allowed alreadyexists=maybe
Success! Data written to: secret/infra/network/allowed
```

I got blocked from creating a path inside of compute, and I didn't need `secret/infra/network` created before making a child path. That infraadmin account is really not needed at all. Let's go ahead and try infracompute.

```
$ vault auth d156326d-1ee6-7a93-d9d3-428e2211962d # auth as infracompute
$ vault write secret/infra/compute/obm/idrac/oem username=root password=calvin
Success! Data written to: secret/infra/compute/obm/idrac/oem
$ vault read secret/infra/compute/obm/idrac/oem
Error reading secret/infra/compute/obm/idrac/oem: Error making API request.

URL: GET http://vaultcore01.pool.lab.ytnoc.net:8200/v1/secret/infra/compute/obm/idrac/oem
Code: 403. Errors:

* permission denied
```

Oh my. I gave myself create, but not read permissions.  New policies.

infranetwork.hcl
```
path &quot;secret/infra/network/*&quot; {
  capabilities = [&quot;create&quot;, &quot;read&quot;, &quot;update&quot;, &quot;delete&quot;, &quot;list&quot;]
}

path &quot;secret/infra/compute/obm/*&quot; {
  capabilities = [&quot;read&quot;, &quot;list&quot;]
}

path &quot;auth/token/lookup-self&quot; {
  capabilities = [&quot;read&quot;]
}
```

infracompute.hcl
```
path &quot;secret/infra/compute/*&quot; {
  capabilities = [&quot;create&quot;, &quot;read&quot;, &quot;update&quot;, &quot;delete&quot;, &quot;list&quot;]
}

path &quot;auth/token/lookup-self&quot; {
  capabilities = [&quot;read&quot;]
}
```

Let's update our policy list and cleanup.

```
vault auth b6eac78d-f278-4d32-6894-a8168d055340 # auth as root token
vault policy-delete infraadmin # delete unneeded infradmin policy
vault token-revoke d16dd3dc-cd9e-15e1-8e41-fef4168a429e # remove infraadmin token
vault policy-write infranetwork infranetwork.hcl
vault policy-write infracompute infracompute.hcl
```

Try again:

```
$ vault auth d156326d-1ee6-7a93-d9d3-428e2211962d # auth as infracompute
Successfully authenticated! You are now logged in.
token: d156326d-1ee6-7a93-d9d3-428e2211962d
token_duration: 2762315
token_policies: [default infracompute]
$ vault read secret/infra/compute/obm/idrac/oem
Key             	Value
---             	-----
refresh_interval	768h0m0s
password        	calvin
username        	root
```

And as network

```
$ vault auth 84faa448-20d9-b472-349f-1053c81ff4c9 #infranetwork
$ vault list secret/infra/compute
Error reading secret/infra/compute/: Error making API request.

URL: GET http://vaultcore01.pool.lab.ytnoc.net:8200/v1/secret/infra/compute?list=true
Code: 403. Errors:

* permission denied
$ vault list secret/infra/compute/obm
Keys
----
idrac/

$ vault list secret/infra/compute/obm/idrac
Keys
----
oem

$ vault read secret/infra/compute/obm/idrac/oem
Key             	Value
---             	-----
refresh_interval	768h0m0s
password        	calvin
username        	root
```
