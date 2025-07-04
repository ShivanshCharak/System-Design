# Scaling
 - In order to  server multiple request u need to have required amount of resources or compute needed 

 ## Two Scaling Strategies
 ### Vertical Scaling

- Making the infrastructure bulikier add more CPU, RAM, DISK
- Easy to manage
- Risk of downtime
- **Fails after a point**:  Infinite vertical scaling is not possible becuase hardware issues/limitation 

### Horizontal Scaling
- Complext structure
- Linear amplification: Adding server is easy for example if you know your server is sevring 100 reqest per secind and now you want to sevrre 500 request per second so u just need to spin up 4 more EC2 instances
- Fault tolerance

##

- For Every Application Scale it vertically first then horizonatlly
Horizaonral scalling is alomst infinitely scalable but theres is a catch

- TO Make sure that your DB and cache can handle these many concurrent reuqest assign them that many resoures so they can respond to it accordingly
- Hence whenever you scale , scale it bottom up 

- If you want to scale api serveres , scale db first


### Scaling DBS
- There are three ways usally verical scaling i.e add more ram and disk
- Create Read replicas 
- SHardin i.e diving the data to multiple dbs thru hashing

