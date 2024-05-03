# Load Balancing vs Application Load Balancing

If you have ever used any sort of Load Balancer in AWS, Azure or maybe you're cool and used Kubernetes CRM to create objects like Ingress or LoadBalancer service, you might've realized there is not one type of load balancer, apparently there are mainly two type of Load Balancers, one being Application Load Balancer, and other being the Classic Load Balancer.

Comparision in brief:

| Functionality            | Classic Load Balancers                | Application Load Balancers          |
|--------------------------|---------------------------------------|------------------------------------|
| Layer of Operation | Layer 4 (Transport Layer) | Layer 7 (Application Layer) |
| Routing | Based on IP addresses and port numbers | Based on application-level data |
| Load Balancing Algorithms | Basic algorithms (round-robin, least connections) | Advanced algorithms (content-based routing, SSL termination) |
| Processing Requirements | Minimal, Hence highly performant | Moderate to High |
| Support for Complex Apps | Limited | Comprehensive |
| Security Features | Limited | Enhanced (content-based firewalling, DDoS protection) |

But even after looking at this table, it bugged me for days, why ingress object allows me to do path based or host based routing on kubernetes, but not the loadbalancer serivce in kubernetes? 

Remember the OSI Model?

What happens is, at Layer 4 the **packet** (from Layer 3) which is now known as a **segment** looks something like this:

| Field             | Size (bytes) | Description                                                   |
|-------------------|--------------|---------------------------------------------------------------|
| Source Port       | 2            | Identifies the sender's port number                           |
| Destination Port  | 2            | Identifies the recipient's port number                        |
| Sequence Number   | 4            | Specifies the sequence number of the first data byte in this segment |
| Acknowledgment Number | 4        | Indicates the next sequence number the sender of the segment is expecting to receive |
| Data Offset       | 4 bits       | Specifies the number of 32-bit words in the TCP header         |
| Reserved          | 6 bits       | Reserved for future use                                        |
| Flags             | 6 bits       | Various control flags such as SYN, ACK, FIN, RST, etc.        |
| Window Size       | 2            | Indicates the size of the sender's receive window             |
| Checksum          | 2            | Provides error-checking for the header and data fields         |
| Urgent Pointer    | 2            | Indicates the end of the urgent data (if present)             |
| Options           | Variable     | Optional field for additional TCP options                      |
| Padding           | Variable     | Padding to ensure the header ends on a 32-bit boundary        |
| Data              | Variable     | Actual data being transmitted                                  |

But at Layer 7, the data is called a message, which looks something like this: (e.g. a Request Message)
                                    
| **Request Line**| Description                                                 |
|-----------------|-------------------------------------------------------------|
| Method          | GET, POST, PUT, DELETE, etc.                                |
| URI             | Uniform Resource Identifier                                  |
| HTTP Version    | Version of the HTTP protocol being used                      |

| **Headers**     | Description                                                 |
|-----------------|-------------------------------------------------------------|
| Host            | Specifies the host and port number of the server             |
| User-Agent      | Identifies the client making the request                     |
| Accept          | Specifies the media types that are acceptable for the response |
| Content-Type    | Specifies the media type of the request body (for POST and PUT requests) |
| Others          | Additional headers as required                               |

| **Body**        | Description                                                 |
|-----------------|-------------------------------------------------------------|
| Content         | Contains the data being sent with the request (if applicable) |
| Example         | Form data, JSON payload, or file content                    |

*The three tables above are part of a single message.*

Did you see that there is no mention of Host, Path or Method at Layer 4? Well, as you can see there isn't, so how is it supposed to even do the fancy load balacing that Application Load Balancer does? Exactly. While the Classic Load Balancers excel at efficiently distributing traffic based on IP addresses and port numbers, Application Load Balancers route requests based on application-level data. I realized my stupidity and went on with my life with this new **obvious** knowledge in my head, which probably should've been there already.

Though the naming convention of AWS still bugs me. Why do they call Layer 4 Load Balancer "Network Load Balancer"? If it is called **Network Load Balancer** because it is balancing traffic for the Layer 3 aka the network layer, then shouldn't Application Load Balancer be called **Presentation Load Balance**? Anyhoo I've been running my single threaded brain too much. Hope you enjoyed the blog.