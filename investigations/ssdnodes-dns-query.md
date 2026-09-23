# DNS Query Investigation — ssdnodes.com

## Trigger
The domain name came through a checkup in DNS record and as there were no initial call to this domain that I was aware of, this became a point of interest to investigate.

## Hypothesis
Since the request came a mobile device on LAN, initial though was around an app or cookies request in that device, whether this is a legitimate request or third-party cookies needs to be investigated.

## Investigation Steps
To start with check to see what did Pi-Hole did with this request, to see where to go next. The request was forwarded to upstream DNS and got answer back. The next step was to see who is DNS holder which in this case it was Cloudflare. And ultimately checking who is the domain holder and any history about this domain. Checking with the domain in website who do security checks such as ssltrust.com.au 

## Findings
The fact that the time to response to this enquiry was 58 ms shows that it followed the usual path in DNS resolving and the DNS provider in this case was Cloudflare.
The next step in knowing the owner, didn’t provide the full information as the owner of the domain locked-down the details in their who is, which is a usual practice in big corporate to limit fishing and exposing unnecessary information in public. However, the fact that this domain was registered back in 2011 can be a positive sign toward the legitimacy of this domain. Upon checking the website on ssltrust.com.au it showed no sign and history of malicious activity by the domain and majority of antiviruses also gave green tick to this website. 

## Conclusion
Although unsure on who initiated this request on the client device, and upon the result on investigation, it is safe to assume this website as a legitimate website and safe to surf. 
