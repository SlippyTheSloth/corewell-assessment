# corewell-assessment

# Assessment
This infrastructure consists of three primary applications that work together to process and store financial data.

**Application #1** is a Windows MSSQL Server that allows financial analysts to login via RDP and upload spreadsheets of data that are processed and output as a JSON file to the attached Data drive.

**Application #2** is a Linux web server comprised of a load balancer, two EC2 instances, and a MariaDB database. Analysts can take the JSON file from Application #1, upload it to Application #2, and process these transactions by visiting financials.example.com. This sends the JSON data to the next application.

**Application #3** consists of a REST API and a MSSQL DB. The JSON file from the web server is recieved by the API, strips out unnecessary data, and sends it to the MSSQL DB where it is stored permanantly.

# Design
I mostly followed the original design due to time constraints. However, I made some extensive changes regarding Application #2. First, I used an Auto-scaling Group for the web server to improve resiliency and scalability. Second, I replaced the two MariaDB EC2 instances with an RDS and ElastiCache running MariaDB. The RDS directly connects to the web server exactly the same as in the original architecture while being more resilient, performant, cheaper, and far easier to manage than a pair of EC2 instances running a custom AMI. The RDS consists of two instances: a primary writable instance connected to web-1 and a read replica connected to web-2. Web-1 can update the DB as usual and both servers can still read from the DB, so no functionality has been altered. Given more time, I would have used Auto-scaling Groups for all of the servers and leveraged AWS Session Manager to improve security when it comes to user access.

# Implementation
This application is deployed via a CloudFormation template. You can import the template using the AWS CloudFormation console, or by using the AWS SAM CLI. Instructions on how to install AWS SAM can be found [on this page.](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html#install-sam-cli-instructions) 

To validate the template, run `sam validate --region us-east-1`

To build the template, run `sam build --region us-east-1`

To deploy the template, run `sam deploy --guided`

# Optimization
All resources are connected to CloudWatch, Compute Optimizer, and Cost Optimizer. Unfortunately I am limited to a free tier account and as such I tested the infra on minimal hardware, so those tools didn't provide much in this constrained situation. That said, they've helped me reduce costs in the past when working in an enterprise setting. My immediate recommendation is to use Auto-scaling Groups rather than individual instances since that lets you adjust your compute resources according to demand. Another easy cost-saving win is to leverage Instance Scheduler to only activate these resources when needed. The design document mentions says the application is only used once per quarter, so everything can be shut down or left in a maintenance state for 90% of the year. Finally, if it takes 10 minutes to process a spreadsheet with 4CPU cores and 8GB of RAM, and you have 8 hours to process 5 spreadsheets, you can *probably* get away with an instance half or a third the size.

# Documentation
There isn't much more to add so I'll take this moment to state the painfully obvious: this repo doesn't create working infrastructure. At first I started working in Terraform, but given the relatively complex architecture and short timeframe I needed to be able to test changes at a faster pace - particularly when it came to networking. I wanted to build this right and that meant minimizing permissions whenever possible and making changes to the design according to best practices. So I decided to build everything in the AWS console where I could quickly test changes, use CloudFormation's IaC generator to create a template with all my resources, make some final edits with the Application Composer, and build the finalized template with AWS SAM. Unfortunately the IaC generator ran into a variety of problems - even making the AWS console slow to a crawl for more than an hour - and ultimately I ended up with a template that lays out the 80+ components in my architecture but failed to import the exact resource IDs for them. This means I was left to manually edit all the references in the template, and I simply ran out of time before the task was complete.

Lesson learned - don't try to use fancy new tools and hope they work as expected when you're strapped for time! If you still have any interest, I will gladly finish this project with an extended deadline. Thank you for the opportunity!
