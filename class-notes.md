# CTF Notes

## During the event

Getting docker images to the Amazon ECR, [one-liner](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-authenticate-registry):

```bash
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
```

where `$AWS_ACCOUNT_ID` is the one you get when running `$ aws sts get-caller-identity --profile team43`, not the Account ID given to us in the private Discord team chat! :facepalm:

---

Programmatic docker build and push:

```bash
docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/team43-$(basename $PWD) .
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/team43-$(basename $PWD):latest
```

---

All-in-one encoding calculator: <https://md5calc.com/> (must-have!)

---

Set kubectl context to avoid writing `-n team43` all the time:

```bash
kubectl config set-context team43 --namespace=team43 \
  --cluster=$CLUSTER_NAME \
  --user=$USER_NAME
kubectl config use-context team43
```

Following [this tutorial](https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/#create-pods-in-each-namespace).

---

Image name from JsonPath

`kubectl get pods users-api-7cf4577b49-mdhxl -o=jsonpath='{.spec.containers[0].image}'`

---

I missed this when skimming the docs for secrets:

> The name of a Secret object must be a valid DNS subdomain name. You can specify the data and/or the stringData field when creating a configuration file for a Secret. The data and the stringData fields are optional. **The values for all keys in the data field have to be base64-encoded strings.** If the conversion to base64 string is not desirable, you can choose to specify the stringData field instead, which accepts arbitrary strings as values.

Proceeded to get stuck for a while on the "take the value of the key JWT_SECRET..." flag.

Also, always remember to `echo -n` when piping strings to encoders like base64.

---

Renan's explanation about attaching a docker shell to the cluster and how addressing works when inside:

> Run the following command:
>
> ```bash
> kubectl -n <TEAM_NAME> run -i -t curl --image=curlimages/curl --restart=Never --command sh
> ```
>
> [...]
>
> Here's what's just happened: when you ran the command kubectl run, Kubernetes span up a new container using the image curlimages/curl (which has the command-line tool curl installed). And because you specified your team's namespace, the container is running on your namespace.
>
> In your Kubernetes cluster, once you deploy a service, the Pods behind the service are accessible via a DNS name: `<service-name>.<namespace>.svc.cluster.local`. To access the Front-end, for example, you'd send a request to the DNS name: `frontend.<TEAM_NAME>.svc.cluster.local`.
>
> Now, you don't necessarily need `svc.cluster.local` since Kubernetes already knows that frontend is the name of a Service. This means you can simply send a request to the DNS name: `frontend.<TEAM_NAME>`. To make it even better, because your curl Pod is running on your team's namespace, you don't even need the namespace in the DNS name. Therefore, you can simply send a request to frontend. Nice, isn't it?!
>
> Before you send a request to frontend, pay attention to one thing: when you wrote the configuration file for the Service, you specified a port (i.e. `<PORT>`). This means you will have to send a request to: `frontend:<PORT>`.

---

How to change a request's `Host` header (when you have an ingress that does name-based virtual hosting):

My way: Firefox lets you edit a request's headers and resend it ([stackoverflow](https://stackoverflow.com/a/26848256)). It's complicated though.

Renan's way:

- Dig the IP address for the load balancer's domain name (which you get with `$ kubectl get ingress`):
  `dig a082d4042562d4cc3adf6ca45a5992c1-a40a28ae7e39a625.elb.us-east-1.amazonaws.com`
- Add a line to `/etc/hosts` like:
  `<IP_ADRESS> frontend.team43.k8s-ctf.com`
- Then make the request from your browser and profit
- And don't forget to remove the `/etc/hosts` line afterwards!

## Thoughts from the closing briefing

Having teammates is probably good, so long as you have the environment set up for it. Unlike me on the Docker CTF.

It's good because of the benefits of pair programming. You have a second (or more) pair of eyes that will stop you on your tracks when you're making a silly mistake, that will help troubleshooting, etc.

They'll also be able to look for information/leads in parallel with you. And when several challenges are concurrent, like 2-3 of the sections here, you can have at those in parallel. Though I guess you notice the concurrency at around the 2nd/3rd challenge of the section.

But for that to work, there are at least two requirements:

- Environment set up, as I mentioned before. That means you should be able to share your screen, and have working voice channels (no need for video aside from the screen-sharing really). I suppose that's why the organizers insisted so much on teams getting together before the event and making sure that all systems were in place.
- If you want to really participate in the CTF, to try solving the challenges yourself rather than follow along as a professional does it, the team-up should be with people with a similar level to you. Which in these CTFs (DevOps, Systems, Security) means my teammates should be noobs like me.

ETA: And ideally only 1-2 people more, because switching drivers will be more agile, and you don't want to be an observer most of the time, you want hands-on experience. On that line, a perk of going solo is that repetition (even if copypasted) helps you drill the concepts and syntax.

Let's strive to do things better next time.

## Further reading

- [the writeup by CyberSecFaith](https://cybersecfaith.com/2021/02/21/devslop-kubernetes-ctf-writeup/) (from the org, QA-ed the ctf)
- [Renan's walkthrough videos](https://www.youtube.com/playlist?list=PLOrvCYDpcF2LH-9kX-IDlegTBGihVeKzs)
