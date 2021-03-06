---
title-prefix: Six Feet Up
pagetitle: SaltStack Training
author: Calvin Hendryx-Parker, CTO, Six Feet Up
author-meta:
    - Calvin Hendryx-Parker
date: Purdue SaltStack 2020
date-meta: May 5, 2020
keywords:
    - Python
    - Orchestration
    - Configuration Management
    - SaltStack
---

# Master & Minions {.semi-filtered data-background-image="images/snake.jpg"}
## Or The Dream of Python Automation
#### Calvin Hendryx-Parker, CTO
#### Six Feet Up

::: notes
* Six Feet Up
* IndyPy
* IndyAWS

Python, Plone, Django
:::

# Certain things should be hand crafted. {data-background-image="https://c2.staticflickr.com/6/5080/7219645500_f79320f7ec_o.jpg"}

::: notes
 * Beverages
:::

# But Not Your Infrastructure {.center .semi-filtered data-background-image="https://c2.staticflickr.com/4/3727/10907456754_2c118f5584_o.jpg"}

::: notes
  But the age of handcrafted servers is over.
  The term bespoke should be reserved for clothing and not our servers that run an application.
:::

# Beautiful Unique Snowflakes are not reproducible. {.semi-filtered data-background-image="https://c2.staticflickr.com/4/3802/12006571534_7a81e7656e_o.jpg"}

::: notes
  tidbit of trivia, the chances of a snowflake being exactly alike is 1 in 1 million trillion (that's a 1 followed by 18 zeros)
:::

# Pets {data-background-image="https://c2.staticflickr.com/6/5233/5828169497_dff82c6543_o.jpg"}

::: notes
coined by Randy Bias after hearing a talk from Bill Baker on Scaling SQL Server
:::

# Cattle {data-background-image="https://c1.staticflickr.com/9/8131/8714106252_4e66254faa_o.jpg"}

::: notes
We are now in the world of DevOps and many non-cloud native applications have the ability to be deployed with cloud native tools.
 How do we take a beautiful handcrafted bare-metal server running our application to translate that into the cloud?
:::

# Rules of DevOps Club {data-background-image="https://cdn-images-1.medium.com/max/2000/1*jEuin1LmfGAMnYPwplzQZg.png"}

* The first rule of DevOps Club is: You do not log into servers
* The second rule of DevOps Club is: You do not log into servers
* Third rule of DevOps Club: If your deployment fails, rollback
* Fourth rule: All artifacts will be stored in source control
* Fifth rule: Only one deployment at a time
* Sixth rule: No one offs, No special cases
* Seventh rule: Deployments will go on as long as they have to
* And the eighth and final rule: If this is your first night at DevOps Club, you have to push to prod.

::: notes
Credit to [Hacker News Post](https://news.ycombinator.com/item?id=8580906) on the subject for inspiration.

For an actual interesting take on being a welcoming DevOps club, make sure to check out [Bridget Kromhout's Post](https://bridgetkromhout.com/blog/the-first-rule-of-devops-club://bridgetkromhout.com/blog/the-first-rule-of-devops-club/) on the subject.
:::

# Tools {data-background-image="https://c1.staticflickr.com/7/6138/5980242616_6ce0a412b0_o.jpg"}

To enter the DevOps world, you need to know what tools are available to you.

Python has many great tools available to use such as SaltStack and the AWS Boto3 library.

# From the Closet to the Cloud {data-background-image="https://c2.staticflickr.com/8/7063/13991487412_7d4a652f38_o.jpg"}

::: notes
College of Engineering at Notre Dame has been hosting their main web site on a single server in a closet on campus since we deployed it for them back in 2011.

In 2016 the university mandated a move to AWS campus wide.

Perfect opportunity to take this aging single server hosting their site and improve their performance and resilience to potential failure.
:::

# Single Server Monolith

``` {.stretch .small}
┌────────────────────────────────────┐
│          Web/App/Database          │
│    ┌───────────────────────────┐   │
│    │           Nginx           │   │
│    └───────────────────────────┘   │
│                                    │
│    ┌───────────────────────────┐   │
│    │          Varnish          │   │
│    └───────────────────────────┘   │
│                                    │
│    ┌───────────────────────────┐   │
│    │          HAProxy          │   │
│    └───────────────────────────┘   │
│                                    │
│    ┌───────────────────────────┐   │
│    │    Plone Instances x 4    │   │
│    └───────────────────────────┘   │
│                                    │
│    ┌───────────────────────────┐   │
│    │    ZEO Storage Server     │   │
│    └───────────────────────────┘   │
│                                    │
└────────────────────────────────────┘
```

# Cloud Optimized {data-background-image="https://c2.staticflickr.com/6/5484/11423341454_b51b661bc5_k.jpg"}

:::::::::::::: {.columns}
::: {.column width="60%"}
:::
::: {.column width="40%"}
![](images/ND AWS Proxy Chain.png){.plain height="500"}
:::
::::::::::::::


::: notes
But how do we make it repeatable.
:::

# Automation {data-background-image="https://c1.staticflickr.com/9/8662/16405965448_b06735c9f7_o.jpg"}

# Enter SaltStack and Boto3 {data-background-image="https://c2.staticflickr.com/4/3025/2302759066_84935ab458_o.jpg"}

# Network End Goal

``` {.stretch .smaller}
┌─────────────────────────────────────────┐                ┌────────────────────────────────────┐
│               control-vpc               │                │              prod-vpc              │
│  ┌───────────────────────────────────┐  │                │  ┌───────────────────────────┐     │
│  │     control-public-az1-subnet     │  │                │  │                           ├─┐   │
│  │                                   │  │                │  │                           │ │   │
│  │                                   │  │                │  │  prod-public-az1-subnet   │ │   │
│  │                                   │  │                │  │                           │ │   │
│  │       ┌───────────────────┐       │  │                │  │                           │ │   │
│  │       │                   │       │  │   Security     │  └┬──────────────────────────┘ │   │
│  │       │  salt.esc.nd.edu  │       │  │────Group ──────│   └────────────────────────────┘   │
│  │       │                   │       │  │    Rules       │  ┌────────────────────────────┐    │
│  │       │  Salt Master/VPN  │       │  │                │  │                            ├─┐  │
│  │       │                   │       │  │                │  │                            │ │  │
│  │       └───────────────────┘       │  │                │  │  prod-private-az1-subnet   │ │  │
│  │                                   │  │                │  │                            │ │  │
│  │                                   │  │                │  │                            │ │  │
│  └───────────────────────────────────┘  │                │  └┬───────────────────────────┘ │  │
└─────────────────────────────────────────┘                │   └─────────────────────────────┘  │
                     │                                     └────────────────────────────────────┘
                     │                                                                           
                     │                                                                           
                     │                                     ┌────────────────────────────────────┐
                     │                                     │              test-vpc              │
                     │                                     │  ┌───────────────────────────┐     │
                     │                                     │  │                           ├─┐   │
                     │                                     │  │                           │ │   │
                     │                                     │  │  test-public-az1-subnet   │ │   │
                     │                                     │  │                           │ │   │
                     │                                     │  │                           │ │   │
                     │             Security                │  └┬──────────────────────────┘ │   │
                     └──────────────Group ─────────────────│   └────────────────────────────┘   │
                                    Rules                  │  ┌────────────────────────────┐    │
                                                           │  │                            ├─┐  │
                                                           │  │                            │ │  │
                                                           │  │  test-private-az1-subnet   │ │  │
                                                           │  │                            │ │  │
                                                           │  │                            │ │  │
                                                           │  └┬───────────────────────────┘ │  │
                                                           │   └─────────────────────────────┘  │
                                                           └────────────────────────────────────┘
```

# Bit of a Chicken and Egg Problem {data-background-image="https://c2.staticflickr.com/8/7508/16136177877_b59f6a4585_k.jpg"}

::: notes
How do we get a fresh Salt master into a region to start building the rest of the infrastructure.

SaltStack has support for various cloud providers, but the easiest was to just use Python and Boto3 to bootstrap the process.

A few lines of Python and you have a VPC ready to be your command and control in any region.
:::

```python
# VPC
c = boto.vpc.connect_to_region(region)
vpc = c.create_vpc('172.20.0.0/16')
vpc.add_tag('Name', 'control-vpc')
c.modify_vpc_attribute(vpc.id,enable_dns_hostnames=True)
```

::: notes
Protip: learn how to securely manage your AWS credentials, no one wants to be a hacker news headline.

Six Feet Up uses LastPass and it can be a poormans secret store for API keys. Check out [lpass-env](https://github.com/luketurner/lpass-env)
:::

# Tool Tip {data-background-image="https://live.staticflickr.com/4709/39538771734_888800273d_k.jpg"}
## Keep your Secrets... Secret

```sh
$ lpass login                  # LastPass
$ $(lpass-env export 'AWS SFU Calvin')
```

```sh
$ eval $(op signin sixfeetup)  # 1Password
$ awsenv "AWS SFU Calvin"
```

```sh
$ env | grep AWS
AWS_ACCESS_KEY_ID=DEADBEEFCAFE
AWS_SECRET_ACCESS_KEY=123jasdfadsOof9akayo5peey0cow
AWS_DEFAULT_REGION=us-east-1
```

::: notes
Would be preferable to use Vault, but if you don't have it, not reason to not be safe
:::

# Bootstrap Continued {data-background-image="https://c1.staticflickr.com/3/2594/4155223042_87b5663358_o.jpg"}

```python
# EC2
data_path = os.path.join(os.path.dirname(__file__), 'bootstrap-master.sh')
with open(data_path) as script:
    bootstrap = script.read()

master = ec2.run_instances(
    'ami-55ef662f',
    subnet_id=subnet.id,
    security_group_ids=[group.id],
    key_name=ssh_key,
    instance_type='t2.micro',
    user_data=bootstrap
)

master.instances[0].add_tag('Name', 'Salt Master')
```

::: notes
From here we make the initial network and a fresh EC2 instance to be our Salt Master in the cloud for this project.

We make use of the `user_data` attribute to pass in a small shell script to get the master up and running.

With one command, we are up and running and ready to really use Salt to automate our environment creation.

Protip: You will have to sometimes put in some wait time in your scripts. For example, you can't associate an Elastic IP right away due to the asynchronous nature of AWS.
:::

# What is SaltStack? {data-background-image="https://c2.staticflickr.com/6/5293/5498122887_fe0d1cfa97_b.jpg"}

::: notes
Event driven orchestration platform written in Python.
:::

# What sets Salt apart? {.semi-filtered data-background-image="images/bg-community-hero3x.original.png"}

* Remote Execution
* Event-Driven Orchestration
* Agent or Agent-less Operation
* Cloud Provisioning
* Speed and Scalability

::: notes
Combines many tools such as:
    * chef/puppet
    * fabric for orchestration
    * terraform for cloud provisioning
:::

# Lonely Minions {.semi-filtered data-background-image="images/depressed_minions.jpg"}

# Master and Minions {data-background-image="images/bello.jpg"}

# Now We Orchestrate

```sh
$ salt-run state.orchestrate orch.deploy-environment pillarenv=prod
```

And build a `test` environment

```sh
$ salt-run state.orchestrate orch.deploy-environment pillarenv=test
```

# Infrastructure (sans humans)

:::::::::::::: {.columns}
::: {.column width="70%"}
```{.stretch .smaller} 
┌─────────────────────────────────────────┐                ┌────────────────────────────────────┐
│               control-vpc               │                │              prod-vpc              │
│  ┌───────────────────────────────────┐  │                │  ┌───────────────────────────┐     │
│  │     control-public-az1-subnet     │  │                │  │                           ├─┐   │
│  │                                   │  │                │  │                           │ │   │
│  │                                   │  │                │  │  prod-public-az1-subnet   │ │   │
│  │                                   │  │                │  │                           │ │   │
│  │       ┌───────────────────┐       │  │                │  │                           │ │   │
│  │       │                   │       │  │   Security     │  └┬──────────────────────────┘ │   │
│  │       │  salt.esc.nd.edu  │       │  │────Group ──────│   └────────────────────────────┘   │
│  │       │                   │       │  │    Rules       │  ┌────────────────────────────┐    │
│  │       │  Salt Master/VPN  │       │  │                │  │                            ├─┐  │
│  │       │                   │       │  │                │  │                            │ │  │
│  │       └───────────────────┘       │  │                │  │  prod-private-az1-subnet   │ │  │
│  │                                   │  │                │  │                            │ │  │
│  │                                   │  │                │  │                            │ │  │
│  └───────────────────────────────────┘  │                │  └┬───────────────────────────┘ │  │
└─────────────────────────────────────────┘                │   └─────────────────────────────┘  │
                     │                                     └────────────────────────────────────┘
                     │                                                                           
                     │                                                                           
                     │                                     ┌────────────────────────────────────┐
                     │                                     │              test-vpc              │
                     │                                     │  ┌───────────────────────────┐     │
                     │                                     │  │                           ├─┐   │
                     │                                     │  │                           │ │   │
                     │                                     │  │  test-public-az1-subnet   │ │   │
                     │                                     │  │                           │ │   │
                     │                                     │  │                           │ │   │
                     │             Security                │  └┬──────────────────────────┘ │   │
                     └──────────────Group ─────────────────│   └────────────────────────────┘   │
                                    Rules                  │  ┌────────────────────────────┐    │
                                                           │  │                            ├─┐  │
                                                           │  │                            │ │  │
                                                           │  │  test-private-az1-subnet   │ │  │
                                                           │  │                            │ │  │
                                                           │  │                            │ │  │
                                                           │  └┬───────────────────────────┘ │  │
                                                           │   └─────────────────────────────┘  │
                                                           └────────────────────────────────────┘
```
:::
::: {.column width="30%"}
![](images/ND AWS Proxy Chain.png){.plain height="500"}
:::
::::::::::::::



::: notes
Every aspect of the infrastructure is defined via Salt pillars and states.
:::

# Orchestrating New Code Releases {data-background-image="https://c1.staticflickr.com/1/72/165372268_ff83a02fa7_o.jpg"}

::: notes
Switch gears and talk about CNM
:::

# Zero Downtime Releases {data-background-image="https://c2.staticflickr.com/4/3448/3746595205_fc31ac8012_o.png"}


```yaml
{%- for app_server in app_servers %}
...
# Stop Varnish, indirectly removing this app server from the NetScaler
stop-{{ app_server }}-varnish:
  salt.function:
    - name: service.stop
    - arg:
      - {{ varnish.service }}
    - tgt: {{ app_server }}
...
```

# Prepare for the Release {data-background-image="https://c2.staticflickr.com/4/3448/3746595205_fc31ac8012_o.png"}

```yaml
...
# Stop Instances
stop-{{ app_server }}-instances:
  salt.function:
    - name: supervisord.stop
    - arg:
      - all
    - tgt: {{ app_server }}
```

# Optional Ad-Hoc Code Releases {data-background-image="https://c2.staticflickr.com/4/3448/3746595205_fc31ac8012_o.png"}

```yaml
...
# Check out a specific revision
{% set branch = salt['pillar.get']('plone:branch', 'dev') %}
# to specify on the command-line: pillar='{"plone": {"branch": "f5d9859"}}'
checkout-code-{{ app_server }}:
  salt.function:
    - tgt: {{ app_server }}
    - name: git.checkout
    - kwarg:
        user: webuser
        rev: {{ branch }}
    - arg:
      - {{ buildout_dir }}
...
```

::: notes
In the CNM environment, we use Salt to talk to the load balancer during releases to remove and add the instances as the code is released to them.
:::

# Handle Release Tasks {data-background-image="https://c2.staticflickr.com/4/3448/3746595205_fc31ac8012_o.png"}


```yaml
# Run the buildout on the app server, but only run the generic setup on one server
run-buildout-{{ app_server }}:
  salt.function:
    - tgt: {{ app_server }}
    - name: cmd.run
    - kwarg:
        cwd: {{ buildout_dir }}
        runas: webuser
    - arg:
      - {{ buildout_dir }}/env/bin/buildout -N
```

# Handle Post Release Tasks {data-background-image="https://c2.staticflickr.com/4/3448/3746595205_fc31ac8012_o.png"}

``` yaml
# Install Gulp bits
install-gulp-{{ app_server }}:
  salt.function:
    - name: npm.install
    - tgt: {{ app_server }}
    - kwarg:
        runas: webuser
        dir: {{ theme_dir }}

# Run the Gulp Build
run-gulp-{{ app_server }}:
  salt.function:
    - tgt: {{ app_server }}
    - name: cmd.run
    - arg:
      - gulp build
    - kwarg:
        runas: webuser
        cwd: {{ theme_dir }}
```

# Not Just for Releases {data-background-image="http://upload.evocdn.co.uk/fruitnet/uploads/asset_image/2_1200434_e.jpg"}

* Migrating Production Data back to Testing
* Scheduled tasks such as Database Backups

# Grabbing Production Data

```yaml
# Find the dev master db for use later
{% set ns = namespace(dev_master=None) %}
{% for dev_master in dev_db_servers
    if salt.saltutil.cmd(dev_master, 'file.search',
        arg=['/var/run/keepalive.state', 'MASTER']).get(dev_master).get('ret') %}
  {% if loop.first %}
    {% set ns.dev_master = dev_master %}
  {% endif %}
{% endfor %}

# Transfer Prod database
{% for prod_slave in prod_db_servers
    if salt.saltutil.cmd(prod_slave, 'file.search',
        arg=['/var/run/keepalive.state', 'BACKUP']).get(prod_slave).get('ret') %}
  {% if loop.last %}

sync-prod-database-to-dev:
  salt.function:
    - name: cmd.run
    - arg:
      - >
        rsync --delete --password-file=/etc/rsyncd.password.dump
        /u02/prodplonedb_archives/ dump@{{ ns.dev_master }}::postgres_dumps
    - tgt: {{ prod_slave }}
  {% endif %}
{% endfor %}
```

::: notes
Be careful of Jinja scoping. If you are in a loop, any variable created in the loop goes away once you exit the loop.

Remember back to our Rules, we want to do everything we can to avoid logging into any servers or creating unique snowflakes.

New Environments can be created in matters of minutes instead of wrestling for days to figure out how something was previously deployed and what handcrafted configurations were put in place.
:::

# Sounds too easy {data-background-image="https://c1.staticflickr.com/1/229/498818720_73a25bdf70_o.jpg"}

The road was bumpy for sure.

Satisfying the rules for no special cases was tricky.

# Mindfulness {.center data-background-image="https://c1.staticflickr.com/5/4397/36076284124_134a37e792_o.jpg"}

> There should be one, and preferably only one, obvious way to do it.
> Although that way may not be obvious at first unless you're Dutch.
> 
-- Tim Peters, The Zen of Python

# The journey operating systems {data-background-image="https://c1.staticflickr.com/5/4902/30826204837_7d745d224f_k.jpg"}

- `2a67758 Editing requirements to run properly on amazon linux`
- `472d844 Refactoring to run CentOS 7 machines`
- `dd67b7a Refactoring for FreeBSD`

What happened here?

::: notes
We needed newer versions of `wv` and `HAProxy` and the repositories included with the Linux distros were old or didn't contain the packages we needed.
:::

# Other Recommendations

* Implement a CI strategy to test your infrastructure with tools like [Kitchen-Salt](https://github.com/saltstack/kitchen-salt)
* Create or use tools to help you trace requests in the cloud <https://github.com/sixfeetup/aws-log-tools>

# Living the Dream {data-background-image="https://i.ytimg.com/vi/B8hrOv2ezag/maxresdefault.jpg"}

::: notes
We are fulfilling the dream, the application runs in a much more Cloud Native Fashion.
:::

# Questions? {data-background-image="https://c1.staticflickr.com/1/92/239595034_d51a99ced1_o.jpg"}

## <calvin@sixfeetup.com>

[`@calvinhp`](https://twitter.com/calvinhp)
