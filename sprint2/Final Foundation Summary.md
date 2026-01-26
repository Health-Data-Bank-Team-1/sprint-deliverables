# Final Foundation Summary

Taking into account client preferences, maintainability, scalability and
compatibility, the backend will be built using a PHP framework called **Laravel**. This
decision was made due to the wide breadth of features and support for the framework
offering simple and robust solutions to all of the core features our project guideline laid
out. Supporting this backend framework will be **MySQL** , as it is a relational database
that the client is familiar with and integrates seamlessly with the chosen framework.

The application will be deployed through a Docker-based containerized
environment, with version control being managed through GitHub.

As for hosting, the DigitalOcean PaaS was selected. This selection allows for the
general concerns and headache of hosting to be pushed off to the platform provider -
meaning an easier time for both us developing the product and for the client
post-handoff. Concerns such as security, scaling, monitoring, updates etc. are all
handled through the platform, ensuring long-term and simple maintainability.

Most importantly, because of the strict security requirements around this project
given the health data being handled, this structure ensures those requirements are met
through multiple layers - Laravel introduces data encryption, RBAC, and audit logging
features, while the PaaS handles security updates, regular encrypted backups etc.
MySQL introduces at-rest encryption and access controls to the data being stored, and
docker containers allow us to create a reproducible environment in order to minimize
any vulnerabilities in deployment.

In short, our final stack will be:

- Laravel (PHP framework)

- MySQL (relational database)

- Docker (containerization)

- DigitalOcean (hosting & managed services)
