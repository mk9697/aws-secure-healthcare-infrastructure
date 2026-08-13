## Private Amazon RDS Deployment

### Decision
RDS was deployed in private database subnets with public accessibility disabled.

### Reasoning
The application server is the only resource that requires database connectivity. There is no business requirement for direct internet access to MySQL.

### Security Impact
This reduces the database attack surface and forces database traffic through approved resources in the application tier.