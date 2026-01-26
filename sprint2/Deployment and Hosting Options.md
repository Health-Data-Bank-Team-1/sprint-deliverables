# Deployment / Hosting Options

Option 1 - VPS (Virtual Private Server) + Web Control Panel

- Very flexible, full compatibility with Laravel
- Runs a cost somewhere in the realm of ~$20 per month
- Access to some partial automated monitoring, backups and security updates via
    the web control panel, but with the caveat that it requires configuration from the
    client
- Git-based deployment
- Web Control Panel means a simple GUI for the usually complex aspects of VPS
    management
- Client will mostly remain responsible for security, recovery, infrastructure
- Offers full control to the client, but will require a level of infrastructure expertise

Option 2 - Platform as a Service (PaaS)

- An option where the only part we worry about is the code of the project, and
everything relating to hosting/deployment is handled by the PaaS (servers,
updates, backups, security).
- Strong Laravel support options (integrated MySQL databases, managed PHP
runtime, environment variable management)
- Git based deployment
- Minimizes server administration concerns
- Very low barrier for client handoff
- One of the more expensive options, ranging from around $30 to as high as $
- Overall lowest barrier for handoff to the client and their ongoing maintenance of
the project. I find this to be the best option in terms of ease on the client, but that
comes with a high recurring cost. Note this also means that the client loses
control over server management, networking etc.

Option 3 - Cloud Hosting (AWS, Azure)

- Full cloud infrastructure, includes managed databases, security, networking
- Full support for Laravel / MySQL, very flexible
- Can deploy through virtual machines, containers
- Access to automated backups, scaling features
- Much higher operational complexity - will require a good level of knowledge of
the infrastructure for both setup and general maintenance. Will require setup of
CI/CD pipelines, access control, networking
- Costs anywhere from the $20-$50 range
- From my research this option would be better reserved for a large-scale
    application, rather than our comparatively small project.

Option 4 - Managed cPanel Hosting

- Medium maintenance burden. Updates and backups for example can be done in
one click (with some more complex caveats such as scheduled jobs requiring an
extra step or composer dependencies being CLI only (important if Laravel is
chosen))
- More cost-effective, running around $15 monthly
- 24/7 support for hosting issues
- Simple handoff in terms of the file / database tasks, but a little more complex for
the framework specific workflows
- Requires workarounds for more modern framework features like queues and
schedulers.
- Decent option but a little more complex than the more expensive options

Option 5 - Shared Hosting

- Cheapest by far, as low as $5 monthly.
- Support for basic, small applications or websites
- Lacks access to tools like Composer, queues or schedulers
- Fully manual deployment
- Minimal automation and monitoring options (client must be very diligent in
manual backup and maintenance management)
- Challenging long-term support
- Generally doesn’t seem to be recommended for modern applications.


## Overall Decision

After combing through the options, I believe choosing a PaaS to be the optimal
choice for this project. PaaS platforms provide an environment for us where concerns
over security, scaling, updates, monitoring etc. are all handled by the platform provider,
meaning the process of deployment will be much easier on us and the process of
maintenance will be much easier on the client. While other options introduce multiple
levels of complexity, a PaaS offers:

- Low maintenance required from the client
- Great security
- Flexible, great scalability
- Git-based deployment
- Dedicated support

Deployment in this manner is simpler than the other options as well - we create a
docker environment, with a github repo for version control. As for which service in
particular, I’ve narrowed it down to two main options after researching - **DigitalOcean**
and **Render**. Both offer similar benefits, with Render seeming to be a little easier to use
and DigitalOcean being generally more cost effective. With this in mind, my
recommendation would be **DigitalOcean** as our platform of choice for deployment of
this project.


