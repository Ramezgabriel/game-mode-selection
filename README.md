# Table of Contents
1. [API of service](#API)
2. [High Level components](#high-level-components)



# API
## swagger-api.yaml
This file has the yaml swagger for the proposed draft API
### Directions to use
Open the https://editor.swagger.io/ web site, copy paste the content of the file on the left hand side of the page and you will be able to view the API on the right hand side. This image is how it would look like

![image](https://user-images.githubusercontent.com/10727531/224495010-722282b5-17a3-4700-853c-85904be0e788.png)

# High level components
The below diagram represents the Interaction between the Actor (the game client on behalf of the player) with the service Game Modes.
![image](https://user-images.githubusercontent.com/10727531/224495359-ec915a33-0d12-417a-a5f0-25ec80b5941c.png)

## API Component
The API component is a component of the service that receives the HTTP calls and responds to the client. It has the following features
- The Component uses API Gateway microservices pattern to be reachable to the out side world
- The Component exposes only publicly accessible routes, even if there are more privately accessible routes, they are not exposed in the same way
- The Component Relies on the API gateway to do the Authentication  (The player is authenticated to make such calls)
- The Component handles the Authorization itself interally (The player is authorized to make such calls)
- The Component Auto scales based on certain criterias

The below is a representation for the Component
![image](https://user-images.githubusercontent.com/10727531/224495773-5906b5ec-4e3a-48d8-95fd-cade67a296f8.png)
NOTE:
- The Load balancer is only accessible to the API gateway to make sure there is a secured way of communciation between the Load balancer and the API gateway.
- The Business Handler is the acutal business logic of the API component and it can only be accessible through the Load balancer

### Concrete Design
It might be benefitial to use the following concrete tech
- API Gateway -> something like nginx or Akamai or AWS API gateway
- Load Balancer -> Application Load balancer from AWS fits the requirment as well as other load balancers in the market
- Business handler -> Most languages work with the Store component chosen, but I believe because of the scale of the service maybe a high level language like java or dotnet is needed. I believe it could be easier to use ECS or EKS to manage the service

One Good option would be to deploy docker containers on AWS EKS for the business handler to be load balanced by AWS ALB

## Store Compoent
The Store component is a component of the service that handles the storage of the data. And the is the only source of truth for the data. It has the following features
- The Component is securily placed so that only the API component can access it
- The Component is managed as a cluster with replication in case of failures
- The Component handles the popularity of modes inheritently to improve performance
- The Component Auto scales based on certain criterias

### Concrete Design
I believe a redis cluster would be a good fit for this usecase. 
It would have Sorted Sets per region with the popularity criteria is based on how many players selects that game mode
The below is a representation of the set for one Region
| Score     | Member      |
| ----------| ----------- |
| 340       | Arena       |
| 523       | 4v4         |

In the above example 4v4 Game mode would be the most popular for the given region

I believe AWS Elasticache would fit the bit. IT has the following features
- Replication in case a failure occurs
- Autoscaling in case we need to have more shards
- It can be placed in a VPC coexisting the Business Handeler of the API component to ensure the access
- Security can be configured using the security groups


## Conrete Architecture
![Concrete](https://user-images.githubusercontent.com/10727531/224496636-8d5b8b9c-3565-42a3-bfdb-b7eb58109d9d.jpg)
