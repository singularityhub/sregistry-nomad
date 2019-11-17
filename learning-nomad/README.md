# Learning Nomad

I have never used nomad, so first I want to follow
the instructions [here](https://learn.hashicorp.com/nomad/getting-started/install)
to learn it, and then will figure out how to do the integration.

## Install and Start Nomad

I first installed from source:

```bash
mkdir -p $GOPATH/src/github.com/hashicorp && cd $_
git clone https://github.com/hashicorp/nomad.git
cd nomad
make bootstrap
```

But found a few issues with dependencies (I had to comment out one of the linting 
commands to install (it was bugging out) and then I was interacting with nomad 
in the local bin of the install folder. And then the regular make dev didn't build
the UI, so I wound up installing a precompiled binary from [this page](https://www.nomadproject.io/downloads.html)

And moving to my bin:

```bash
$ sudo mv nomad /usr/local/bin
```

and then verifying with:

```bash
bin/nomad -v
```

Note that for the following commands, I added this folder to my $PATH and just
referred to "nomad." When we are developing, we'll run a one node "cluster."

```bash
sudo nomad agent -dev
```

In a different terminal you can run:

```bash
$ nomad node statusID        DC   Name                    Class   Drain  Eligibility  Status
4cc26561  dc1  vanessa-ThinkPad-T460s  <none>  false  eligible     ready
```

You can see members of the "gossip ring" like:

```bash
nomad server members
$ nomad server members
Name                           Address    Port  Status  Leader  Protocol  Build       Datacenter  Region
127.0.0.1  4648  alive   true    2         0.10.2-dev  dc1         global
```

## Creating a Dummy Job

Also in another terminal, we can create a job template like this:

```bash
nomad job init
Example job file written to example.nomad
```

Register the example job:

```bash
$  nomad job run example.nomad
==> Monitoring evaluation "f3b62d82"
    Evaluation triggered by job "example"
    Allocation "9d147238" created: node "adaeb7c3", group "cache"
    Evaluation within deployment: "6624b087"
    Evaluation status changed: "pending" -> "complete"
==> Evaluation "f3b62d82" finished with status "complete"
```

and check it's status:

```bash
$ nomad status example
ID            = example
Name          = example
Submit Date   = 2019-11-17T13:01:24-05:00
Type          = service
Priority      = 50
Datacenters   = dc1
Status        = running
Periodic      = false
Parameterized = false

Summary
Task Group  Queued  Starting  Running  Failed  Complete  Lost
cache       0       1         0        0       0         0

Latest Deployment
ID          = 6624b087
Status      = running
Description = Deployment is running

Deployed
Task Group  Desired  Placed  Healthy  Unhealthy  Progress Deadline
cache       1        1       0        0          2019-11-17T13:11:24-05:00

Allocations
ID        Node ID   Task Group  Version  Desired  Status   Created  Modified
9d147238  adaeb7c3  cache       0        run      pending  41s ago  41s ago
```

You can update the file (e.g., change the count or redis version) and then update
like:

```bash
nomad job plan example.nomad # check
nomad job run example.nomad  # run
```

and then stop the job when you finish

```bash
nomad job stop example
```

## Create a Cluster

We can now use the [server.hcl](server.hcl) to start a simple cluster.

```bash
nomad agent -config server.hcl
```

We also have [client1.hcl](client1.hcl) and [client2.hcl](client2.hcl) to use 
for clients.

Then each client needs to be started separately:

```bash
sudo nomad agent -config client1.hcl
sudo nomad agent -config client2.hcl
```

And then you can check the status for both:

```bash
 nomad node status
ID        DC   Name     Class   Drain  Eligibility  Status
f9e74893  dc1  client2  <none>  false  eligible     ready
e4c3ffe2  dc1  client1  <none>  false  eligible     ready
```

We can then submit the same job to the cluster,
check it's status, and then stop it.

```bash
nomad job run example.nomad
nomad status example
nomad job stop example
```

## Web Interface

Just kidding! Start the job again, and then open up to [http://localhost:4646/ui/jobs](http://localhost:4646/ui/jobs)
to see a cool web interface. It should go from status `dead` to `running` when you start it.
Then shut everything down! (Stop the job and press control C for all the agents).

Next I'll check out one of these more [complex example](https://learn.hashicorp.com/nomad/getting-started/next-steps)
and think about how sregistry could fit in. Minimally, the resources that a sregistry
needs could be run as jobs, and then perhaps another for a builder?
