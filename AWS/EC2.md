- reserved instance
- on-demand instance
- spot instance
![[Pasted image 20230208083411.png]]

On-demand instances allow you to pay for compute capacity by the hour or the second, with no long-term commitment. They are well-suited for users that need the flexibility to spin up and spin down instances as needed and are not able to commit to a specific usage pattern.

Reserved instances provide a discount compared to on-demand pricing in exchange for a commitment to use the instances for a 1 or 3-year term. By making a low, one-time payment for each reserved instance, you can save up to 75% compared to the on-demand price.

Spot Instances are ideal for applications that are flexible and can be interrupted, such as big data, containerized workloads, CI/CD pipelines, and scientific computing, among others. When EC2 needs the capacity back, it will interrupt the Spot Instance, giving you a two-minute notice before termination.

Spot Instances provide a cost-effective way to access unused EC2 capacity and can save you up to 90% compared to on-demand pricing. The price for a Spot Instance is determined by the supply and demand for EC2 capacity and can change dynamically, so you need to continually monitor the Spot price and adjust your bid price accordingly.