# Sentia Wordpress Technical Assessment

This repository presents an approach and possibile alternatives to address the assigment [presented here](https://github.com/sentialabs/public-cloud-recruitment/blob/master/ASSIGNEMENT.md)

## Part 1 - Transformation and Migration to the Public Cloud

### Some facts

- The customer is currently hosting 10 wordpress sites using wordpress multisite in a private datacenter.
  - *(10 wordpress instances - How many databases in a multisite environment?)*
- They achieve HA  by using 2 servers and having 2 copies of their multisite
- They are using 2 MySQL servers behind HA proxy to achieve HA

---
### Problem / Opportunity

Some of their websites have increased in popularity especially during certain timeframes

#### Requirements 
- So, the customer has decided to move away from wordpress multisites and have independent Wordpress applications
  - **(10 wordpress instances**
- They have 5 more sites in the making that will reach production in the next 12 months
  - **5 additional sites wordpress instances**

---
### To Do.
#### Design considerations to keep in mind

You have undertaken the task to design the future state of this environment in the public cloud. The solution needs to:

- be scalable and flexible.
- be futureproof and expandable with new WordPress sites with minimal effort.

Please provide a design for the designated architecture in either AWS or Azure. For the same design, please provide CloudFormation or ARM templates, and everything else that you need to accompany your solution with based on your approach.

---

### Approaching the solution

- I decided to address an Azure approach, so if you are curious about all the architecture decisions and technical implementation process, the documentation pricess is [here](https://github.com/bgarcial/sentia-assessment/blob/master/Documentation/README.md)



