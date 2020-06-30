# domain tool

This tool combines a number of AWS API calls to allow the user to interact with some of the *route53*, *route53domains* and *organizations* sub-commands of aws-cli. It was originally developed to copy zone records between AWS accounts. All existing tools required non-AWS API tools which I was not comfortable with using.

The tool greatly simplifies complex operations such as deleting a zone and copying a zone between AWS accounts.

For example, to delete a zone record in Route 53 requires the following calls to the AWS API:

* aws route53 list-hosted-zones
* {convert the output of the above to add "DELETE" as an action}
* aws route53 change-resource-record-sets
* aws route53 wait resource-record-sets-changed
* aws route53 delete-hosted-zone
* aws route53 wait resource-record-sets-changed

This tool wraps all that up with one call: *domain zone-delete*

Copying a zone between AWS accounts requires all manner of gymnastics. This tool does it with two independent calls, *domain zone-show* and *domain zone-create*, which can be combined using a pipe:

```
AWS_PROFILE=profileOne domain zone-show example.com | AWS_PROFILE=profileTwo domain zone-create
```

Alternatively, you can export the zone from one account and import it into another:

```
AWS_PROFILE=profileOne domain zone-show example.com > myZoneFile.json
AWS_PROFILE=profileTwo domain zone-create myZoneFile.json
```

In addition, I noticed that I was often needing to move zones to new-sub accounts. This creates that account, but currently does *NOT* create an IAM user or even activate the account, as a minimum viable product, this tool just creates a sub-account in your organisation. To gain access, attempt to log-in on the console and use the "Forgot Password" option to receive an email from AWS.


## Supported Commands

*account-create* email [account name]
* Create a sub-account with email address and optional name.

*domain-dig* domain-name
* Return the name servers for domain-name as listed on the DNS.

*domain-list*
* List all the domains in this account.

*domain-dns* domain-name
* Display the name servers as configured in this account.

*domain-set* domain-name
* Set the name servers for domain-name to the servers configured in the Route 53 record in this account.

*domain-whois* domain-name
* Display the whois information for domain-name.

*zone-compare* domain-name
* Compare the Route53 domain servers with the DNS.

*zone-create* [filename]
* Create a zone record. Use optional filename as the source or pipe via stdin.

*zone-delete* domain-name
* Delete the zone record for domain-name.

*zone-list* [domain-name]
* Return a list of zone-ids and domain names in this account. If a domain-name is specified, this returns the zone-id of that domain.

*zone-show* domain-name
* Display the zone record for domain-name.

*help*
* Display list of commands.


## Dependencies

This tool is written for bash. It uses the following external tools:

* aws
* cat
* cut
* date
* dig
* grep
* jq
* paste
* sed
* tr
* uuidgen


## Usage

The tool expects the same authentication as the aws-cli tool. If you specify AWS_PROFILE before the tool, it will be used automatically by the aws-cli:

Example: ```AWS_PROFILE=myProfile domain domain-list```

The permissions associated with the API calls are determined by the profile you use. The command itself only uses a very small set of aws-cli calls. All calls to the aws-cli are logged in .timestamp.aws.log files in the current directory (note the leading '.').

If you want to see more detailed output, you can call the tool using the bash -x option, which will turn a DEBUG flag that prevents any data altering aws-cli calls to be executed, instead echoing them to the console. For example: ```bash -x domain domain-list```


## Disclaimer

I use this tool. It works for me. It might not for you. It might kill a kitten when you use it. You've been warned.

It is possible that this tool has assumptions built-in that I'm not aware of that cause you grief. If the tool fails for you, please provide a pull-request.


## Author

Onno Benschop, ITmaze - onno@itmaze.com.au
