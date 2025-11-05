# Video Script: Module 1.2.1.4 - Cloud vs On-Premises

**Duration:** 26 minutes
**Module:** 1.2.1.4
**Prerequisites:** Module 1.2.1.1 (UiPath Orchestrator Architecture), 1.2.1.2 (Authentication)

---

## Section 1: Introduction (2 minutes)

### Visual
- Title slide
- Split screen: Cloud vs On-Premises comparison
- Use case examples

### Script

Welcome to Module 1.2.1.4: Cloud vs On-Premises. In this module, we're going to compare the two deployment models for UiPath Orchestrator—Cloud and On-Premises—and help you understand which one to use for your automation projects.

UiPath Orchestrator can be deployed in two ways: in the UiPath Cloud, where UiPath manages all the infrastructure for you, or On-Premises, where you install and manage Orchestrator on your own servers.

Both deployment models offer the same core Orchestrator functionality—managing robots, processes, queues, and so on. But they differ significantly in how they're managed, how you access them, and which security and compliance requirements they meet.

As a developer working with the Python SDK, you need to understand these differences because they affect how you configure your API clients, how you handle authentication, and sometimes even which features are available.

By the end of this module, you'll understand the characteristics of each deployment model, how to configure your code to work with both, and most importantly—how to choose the right deployment model for your specific requirements.

Let's start by looking at UiPath Cloud.

### Presenter Notes
- Emphasize that both models offer the same core functionality
- Mention that this knowledge is critical for configuration
- Set expectation that we'll provide decision criteria

---

## Section 2: UiPath Cloud Overview (3 minutes)

### Visual
- Cloud architecture diagram
- Benefits list
- Example Cloud URL

### Script

UiPath Cloud is a SaaS—Software as a Service—offering where UiPath hosts and manages Orchestrator for you. You access it via the internet at cloud.uipath.com.

With Cloud, there's zero infrastructure setup. You don't install anything, you don't manage servers, you don't configure databases. You simply sign up, create your organization and tenants, and start using Orchestrator within minutes. This is perfect for companies that want to get started quickly without infrastructure overhead.

UiPath manages everything: the servers, the databases, the network, the backups, the updates. Your IT team doesn't need to worry about infrastructure maintenance. It's a true SaaS model—you just use the service, UiPath handles the rest.

One of the biggest benefits is automatic updates. When UiPath releases new features, they're automatically available in Cloud. There's no upgrade project, no downtime, no testing of new versions—it just happens. You always have the latest features and security patches. This means Cloud users typically get new features months before they're available in On-Premises.

UiPath Cloud is highly scalable. If you suddenly need to go from 10 robots to 100 robots, Cloud can handle it. No need to procure new hardware or worry about capacity planning. The infrastructure scales automatically.

The URL structure for Cloud is specific: cloud.uipath.com, followed by your organization name, your tenant name, and then orchestrator underscore. For example: cloud.uipath.com/acmecorp/production/orchestrator_. This structure includes the organization and tenant names in the URL path, which is different from On-Premises as we'll see.

Cloud is great for companies that want rapid deployment, minimal IT overhead, automatic updates, and easy scaling. It's particularly popular with small to medium businesses, startups, and companies testing RPA before making a large infrastructure investment.

But Cloud does have some constraints. It requires internet connectivity—you can't deploy Cloud in an air-gapped network. And data is stored in UiPath's data centers, which may not meet certain compliance requirements. Let's now look at the alternative: On-Premises.

### Presenter Notes
- Emphasize "zero infrastructure" aspect
- Mention automatic updates as key benefit
- Preview constraints that lead to On-Premises choice

---

## Section 3: On-Premises Overview (3 minutes)

### Visual
- On-Premises architecture diagram
- Benefits list
- Example On-Premises URL

### Script

On-Premises deployment means you install and manage Orchestrator on your own infrastructure—in your data center, on your servers, in your network.

With On-Premises, you have complete control. You choose the servers, you configure the network, you set security policies, you manage everything. This gives you maximum flexibility and customization. You can integrate Orchestrator deeply with your existing systems—Active Directory, your internal databases, your monitoring tools, whatever you have.

On-Premises can be deployed in air-gapped networks—environments with no internet connection. This is critical for highly secure environments like government, defense, or certain financial institutions. With On-Premises, data never leaves your network. You control exactly where data is stored, who can access it, and how it's secured.

You also control when to upgrade. When UiPath releases a new version, you decide when to install it. You can test the new version in your dev environment, validate it, and then schedule the upgrade at a time that works for you. This gives you stability—production doesn't change unless you decide to change it.

The URL structure for On-Premises is simpler: it's just your custom domain. For example: orchestrator.acmecorp.com. No organization or tenant in the URL path—that's determined by authentication. Much simpler than Cloud's URL structure.

On-Premises is ideal for organizations with strict security or compliance requirements, such as data residency laws that require data to stay in specific geographic locations. It's common in banking, healthcare, government, and other highly regulated industries. It's also used by large enterprises with established IT infrastructure and the capacity to manage it.

But On-Premises requires significant IT resources. You need to set up servers, configure networking, manage databases, handle backups, monitor performance, and perform upgrades. All of that is your responsibility. So On-Premises is typically chosen when security, compliance, or control requirements outweigh the IT overhead.

Now let's compare these two models in more detail, starting with API endpoint differences.

### Presenter Notes
- Emphasize complete control and air-gap capability
- Mention typical industries that choose On-Premises
- Acknowledge the IT overhead tradeoff

---

## Section 4: API Endpoint Differences (3 minutes)

### Visual
- Side-by-side code comparison: Cloud vs On-Premises
- URL structure breakdown
- Configuration examples

### Script

Let's look at the API endpoint differences between Cloud and On-Premises, because this directly affects how you configure your Python code.

For Cloud, the base URL includes the organization and tenant names in the path. The format is: cloud.uipath.com, slash your organization, slash your tenant, slash orchestrator underscore, and then slash odata slash the endpoint. For example, if your organization is "acmecorp" and your tenant is "production", the URL for getting robots would be: cloud.uipath.com/acmecorp/production/orchestrator_/odata/Robots.

Notice that "orchestrator" has an underscore at the end—that's not a typo, it's part of the URL structure.

For On-Premises, it's simpler: it's your custom domain, slash odata, slash the endpoint. For example: orchestrator.acmecorp.com/odata/Robots. No organization or tenant in the URL—those are determined by your authentication and which tenant you authenticated to.

The authentication endpoint is also different. For Cloud, it's: cloud.uipath.com/identity underscore/connect/token. For On-Premises, it's: your domain/identity/connect/token. Notice the underscore in Cloud's "identity_" versus no underscore in On-Premises "identity".

These differences might seem minor, but they're critical to get right in your code. If you hard-code a Cloud URL and then need to switch to On-Premises, you'll have problems. Best practice is to use environment variables or configuration files to store these URLs, so you can easily switch between deployments.

Here's a code example. For Cloud, you'd configure:

```python
BASE_URL = 'https://cloud.uipath.com/acmecorp/prod/orchestrator_'
TOKEN_URL = 'https://cloud.uipath.com/identity_/connect/token'
```

For On-Premises:

```python
BASE_URL = 'https://orchestrator.acmecorp.com'
TOKEN_URL = 'https://orchestrator.acmecorp.com/identity/connect/token'
```

Other than the URLs, the API calls are identical. Same headers, same OAuth flow, same OData queries. This makes it relatively easy to write code that works with both deployment models—you just need to abstract the base URLs.

We'll see a complete code example later that shows how to handle both deployment types elegantly.

### Presenter Notes
- Walk through URL structures slowly—students often miss details
- Emphasize the underscore in Cloud URLs
- Mention that this is where many configuration errors happen

---

## Section 5: Authentication Differences (3 minutes)

### Visual
- Authentication flow diagrams for both models
- Code examples showing OAuth flow
- Available authentication methods comparison

### Script

Both Cloud and On-Premises use OAuth 2.0 for API authentication, but there are some important differences in how authentication is configured and managed.

In Cloud, authentication is handled by UiPath's centralized identity service. When you create an External Application in Cloud, you get a client ID and client secret. You use these credentials with the Cloud token endpoint to get an access token. It's a centralized, standardized OAuth 2.0 flow. Everyone using Cloud uses the same authentication mechanism.

Cloud also supports single sign-on with external identity providers like Azure AD, Okta, and others. And multi-factor authentication is available for enhanced security. But from an API perspective, you always use the External Application credentials and the Client Credentials OAuth flow.

On-Premises offers more flexibility in authentication methods. You can use local Orchestrator accounts, Windows Authentication, Active Directory, SAML-based single sign-on, Azure AD, or other custom identity providers. Your IT team configures which authentication methods are available.

For API access, On-Premises also supports OAuth 2.0 with External Applications, similar to Cloud. But the token endpoint is on your domain, and the authentication might integrate with your Active Directory or other internal systems. The OAuth flow is the same—client credentials, client ID and secret, request a token—but it's hitting your On-Premises server instead of UiPath's cloud service.

From a code perspective, the OAuth flow is nearly identical between Cloud and On-Premises. You POST to the token endpoint with your credentials, get back an access token, and use it in your API requests. The main difference is just the URL of the token endpoint.

Here's an example that works for both:

```python
def get_token(token_url, client_id, client_secret):
    response = requests.post(token_url, data={
        'grant_type': 'client_credentials',
        'client_id': client_id,
        'client_secret': client_secret',
        'scope': 'OR.Robots OR.Jobs'
    })
    return response.json()['access_token']

# Use with Cloud
cloud_token = get_token(
    'https://cloud.uipath.com/identity_/connect/token',
    cloud_client_id,
    cloud_client_secret
)

# Use with On-Premises
onprem_token = get_token(
    'https://orchestrator.acme.com/identity/connect/token',
    onprem_client_id,
    onprem_client_secret
)
```

Same function, different URLs. This is why it's important to keep your code flexible and configurable.

### Presenter Notes
- Emphasize that OAuth flow is the same, URLs differ
- Mention that On-Premises gives more authentication options
- Note that code is largely portable

---

## Section 6: Features and Updates (3 minutes)

### Visual
- Timeline showing feature rollout: Cloud vs On-Premises
- Version management comparison
- Feature availability table

### Script

Let's talk about feature availability and how updates work in each deployment model.

With Cloud, UiPath continuously deploys new features. When a feature is ready, it's rolled out to Cloud, often gradually—maybe to a percentage of organizations first, then to everyone. You don't control when this happens—it just appears. You log in one day and there's a new feature available. This means Cloud users get access to the latest capabilities immediately, sometimes months before they're available in On-Premises.

Updates happen automatically, often with zero downtime. UiPath manages the upgrade process, ensures backward compatibility, and handles any issues. From your perspective, you always have the latest version. This is great for getting new features, but it means you don't control the upgrade timing. If a feature changes, you need to adapt.

With On-Premises, new features come with new versions, typically released quarterly or semi-annually. When a new version is released, it's up to you to upgrade. You download the new version, test it in your dev environment, validate that your automations still work, and then schedule a maintenance window to upgrade production.

This gives you complete control over when production changes. If you have a critical period where you can't afford any disruptions, you simply don't upgrade during that time. But it also means you're responsible for the upgrade process, and you might be running on an older version that doesn't have the latest features.

For API developers, this difference matters. If you're using a feature that was recently added to Cloud, it might not be available yet in On-Premises, or it might be in the latest version that your organization hasn't upgraded to yet. You need to be aware of version differences when writing code that needs to work across both deployments.

Best practice: check the API documentation for the minimum required version of Orchestrator for the features you're using. And if you're writing code that will run against both Cloud and On-Premises, test it against the On-Premises version you're targeting, not just against Cloud.

UiPath maintains backward compatibility for a reasonable window—typically the current version plus two prior versions are supported. So if you write code that works with the current On-Premises version, it should continue to work even as Cloud moves ahead.

### Presenter Notes
- Clarify that Cloud is always ahead in features
- Mention the importance of version awareness for On-Premises
- Emphasize testing against target versions

---

## Section 7: Network, Security, and Compliance (3 minutes)

### Visual
- Network diagrams for both models
- Security comparison matrix
- Compliance requirements examples

### Script

Let's discuss network requirements, security, and compliance—these are often the deciding factors in choosing between Cloud and On-Premises.

Cloud requires internet connectivity. Your robots, your API scripts, your developers—everyone needs to be able to reach cloud.uipath.com over HTTPS. This means opening firewall rules, ensuring internet access, and accepting that your Orchestrator is on the public internet. It's secured with OAuth, encryption, and UiPath's security measures, but it's not isolated within your network.

On-Premises can be deployed in completely air-gapped networks with no internet access at all. This is critical for defense, intelligence agencies, certain financial institutions, and other organizations with maximum security requirements. In an air-gapped deployment, Orchestrator is entirely within your private network, with no external connections.

Even when On-Premises has internet access, you control the network configuration. You can set up VPNs, configure specific firewall rules, integrate with your network monitoring, whatever you need. Full control.

From a security perspective, both models are secure, but the security model differs. With Cloud, UiPath is responsible for security. They have SOC 2 and ISO 27001 certifications, regular security audits, and dedicated security teams. Data is encrypted in transit and at rest. For most organizations, this is more than sufficient.

With On-Premises, you're responsible for security. This can be an advantage if you have strict security policies and a skilled security team. You can apply custom security measures, integrate with your security tools, and fully control the security posture. But it's also a responsibility—if something goes wrong, it's on you.

Compliance requirements often drive the decision. Cloud meets most standard compliance needs—GDPR, SOC 2, ISO 27001. And UiPath offers data residency options, allowing you to choose which region your data is stored in—US, EU, Japan, etc.

But some compliance requirements mandate On-Premises. Examples include: FedRAMP for US federal agencies, certain HIPAA interpretations for healthcare data, data localization laws in some countries that require data to physically stay within national borders, and custom compliance requirements in banking and finance.

If your organization is in a highly regulated industry, or if you have data that absolutely cannot leave your network, On-Premises is likely your only option. If you have standard compliance needs and can accept data in UiPath's cloud, Cloud is often simpler.

### Presenter Notes
- Emphasize that air-gap capability is unique to On-Premises
- Clarify that both can be secure, but security models differ
- Mention that compliance often dictates the choice

---

## Section 8: Management and Maintenance (2 minutes)

### Visual
- Management responsibilities comparison
- Maintenance tasks list
- Cost comparison factors

### Script

Let's talk about management and maintenance, which directly impacts IT workload and costs.

With Cloud, UiPath handles everything. Server management, database administration, backups, disaster recovery, performance monitoring, security patches—it's all managed by UiPath. Your IT team doesn't need to worry about infrastructure. The only thing you manage is your Orchestrator configuration—users, roles, processes, robots, and so on.

This translates to significant IT time savings. No need for dedicated Orchestrator administrators, no late-night maintenance windows, no emergency patching. The IT overhead is minimal.

With On-Premises, you manage everything. You need to set up servers, configure load balancing for high availability, manage SQL Server databases, configure backups and disaster recovery, monitor performance, apply Windows updates, apply Orchestrator updates, and manage the network.

This requires skilled IT staff and ongoing effort. You need database administrators, system administrators, and potentially network engineers. You need to plan maintenance windows for updates, and you need 24/7 monitoring to catch issues.

The cost comparison is complex. Cloud has a predictable subscription cost per user or per robot. No infrastructure costs, but you pay monthly or annually for as long as you use it.

On-Premises has software licensing costs—either perpetual licenses or subscriptions—plus infrastructure costs for servers, storage, and networking. Plus IT staff costs for management. The upfront cost is higher, but for large deployments, the long-term cost can be lower than Cloud.

The break-even point varies by organization, but generally, for smaller deployments (under 100 robots), Cloud is more cost-effective. For very large deployments (hundreds of robots), On-Premises can be cheaper over multiple years, assuming you have the IT capacity.

But cost isn't just money—it's also time to value. Cloud gets you up and running in days, On-Premises can take months. That time has value.

### Presenter Notes
- Emphasize IT overhead difference
- Clarify that cost depends on scale and timeframe
- Mention time-to-value as a factor

---

## Section 9: Hybrid Deployments (3 minutes)

### Visual
- Hybrid architecture diagram
- Use case example: Healthcare scenario
- Code example for hybrid setup

### Script

Now let's talk about hybrid deployments, which combine the benefits of both Cloud and On-Premises.

In a hybrid deployment, you use Cloud Orchestrator for management and control, but your robots run on-premises in your network. The robots connect to Cloud Orchestrator over the internet to get job instructions, but they process data locally and never send it to the cloud.

This is perfect for scenarios where you want the ease and low overhead of Cloud management, but you have data that cannot leave your network due to security or compliance requirements.

Let's look at a concrete example: a healthcare organization with patient health information, which is protected by HIPAA regulations. They want to use RPA to automate patient data processing, but PHI—Protected Health Information—cannot leave their network.

The solution: Cloud Orchestrator with on-premises robots. The Orchestrator in the cloud manages the robots, schedules jobs, and collects logs—but only non-PHI data. The robots themselves are installed on servers in the healthcare organization's data center. When a robot processes patient data, it reads from the local database, processes locally, and writes back to the local system. The patient data never leaves the network. Only control messages and non-sensitive logs go to Cloud Orchestrator.

This gives them the benefits of Cloud—easy management, automatic updates, no Orchestrator infrastructure to maintain—while keeping patient data on-premises for compliance.

Another example: a financial institution with customer transaction data. They use Cloud for non-sensitive automation like report generation and monitoring. But for processes that handle actual customer financial data, they use on-premises robots. Same Orchestrator, but sensitive processing stays local.

From an API perspective, hybrid is straightforward. Your API client connects to Cloud Orchestrator—same URLs, same OAuth, same API endpoints. The difference is where the robots are physically located and what data they're allowed to access. Your API code doesn't change.

Here's what the configuration looks like:

```python
# Cloud Orchestrator—same as Cloud deployment
BASE_URL = 'https://cloud.uipath.com/healthcare/prod/orchestrator_'
TOKEN_URL = 'https://cloud.uipath.com/identity_/connect/token'

# API calls go to Cloud
robots = requests.get(
    f'{BASE_URL}/odata/Robots',
    headers={'Authorization': f'Bearer {token}'}
)

# But robots are on-premises, processing data locally
# They connect to Cloud for control, but data stays local
```

Hybrid deployments require careful planning—you need to configure robots to connect to Cloud, set up network rules, and be clear about what data can and cannot leave your network. But for many organizations, hybrid is the ideal balance.

### Presenter Notes
- Emphasize that hybrid combines benefits of both
- Use healthcare example—very relatable
- Clarify that from API perspective, it's just Cloud

---

## Section 10: Decision Framework (2 minutes)

### Visual
- Decision tree flowchart
- Use case examples with recommendations
- Checklist for choosing

### Script

So how do you choose between Cloud, On-Premises, and Hybrid? Let's look at a decision framework.

Start with compliance and security requirements. If you have strict data residency laws, if you need an air-gapped deployment, or if you have compliance mandates that require on-premises infrastructure, then On-Premises is your answer. No decision needed—compliance requirements dictate the choice.

If compliance allows Cloud, next consider your IT capacity. Do you have the skilled staff and resources to manage Orchestrator infrastructure? Are you willing to take on that overhead? If yes, On-Premises is an option. If no, or if you'd prefer not to, Cloud is better.

Next, think about time to value. Do you need to get started quickly, test RPA, or prove value before making a large investment? Cloud is the faster path. On-Premises requires months of setup.

Consider scale. Are you deploying 10 robots or 1,000? For small to medium deployments, Cloud's subscription model is cost-effective. For very large deployments, On-Premises can be more economical long-term, but you need to consider total cost including IT staff.

Think about feature velocity. Do you want the latest features as soon as they're available? Cloud gives you that. Do you prefer stability and controlled upgrades? On-Premises gives you control.

Finally, consider hybrid. If you want Cloud's management benefits but have some sensitive data, hybrid might be the answer.

Here are some example scenarios:

A startup with no IT infrastructure testing RPA: Cloud. Easy choice—quick setup, no infrastructure, low cost to start.

A bank with strict data residency requirements: On-Premises. Compliance drives the decision.

A healthcare organization with HIPAA requirements: Hybrid. Cloud Orchestrator with on-premises robots processing PHI.

A global company with 500 robots and a mature IT department: On-Premises. Scale and IT capacity make On-Premises economical.

A mid-size company in a standard industry: Cloud. Unless there are specific reasons for On-Premises, Cloud is simpler.

Think through these factors for your specific situation, and the right choice usually becomes clear.

### Presenter Notes
- Present as a logical decision process
- Use examples to make it concrete
- Emphasize that compliance often decides

---

## Section 11: Code Examples (3 minutes)

### Visual
- Complete code example supporting both deployment types
- Configuration file example
- Running code demonstration

### Script

Let's look at a complete code example that supports both Cloud and On-Premises deployments through configuration.

The key is to abstract the differences—URL structure and authentication endpoint—so your core logic remains the same.

Here's an approach using a client class:

```python
import os
import requests

class OrchestratorClient:
    def __init__(self, deployment_type='cloud', **config):
        """Initialize client for Cloud or On-Premises"""
        self.deployment_type = deployment_type

        # Set URLs based on deployment type
        if deployment_type == 'cloud':
            self.base_url = config['cloud_base_url']
            self.token_url = config['cloud_token_url']
        else:  # on-premises
            self.base_url = config['onprem_base_url']
            self.token_url = config['onprem_token_url']

        self.client_id = config['client_id']
        self.client_secret = config['client_secret']

        # Authenticate
        self.token = self._get_token()

    def _get_token(self):
        """Get OAuth access token (same for both)"""
        response = requests.post(self.token_url, data={
            'grant_type': 'client_credentials',
            'client_id': self.client_id,
            'client_secret': self.client_secret',
            'scope': 'OR.Robots OR.Jobs OR.Folders'
        })
        response.raise_for_status()
        return response.json()['access_token']

    def get_robots(self, folder_id):
        """Get robots (same API for both)"""
        response = requests.get(
            f'{self.base_url}/odata/Robots',
            headers={
                'Authorization': f'Bearer {self.token}',
                'X-UIPATH-OrganizationUnitId': str(folder_id)
            }
        )
        response.raise_for_status()
        return response.json()['value']

    def start_job(self, process_key, folder_id, robot_ids=None):
        """Start a job (same API for both)"""
        payload = {
            'startInfo': {
                'ReleaseKey': process_key,
                'Strategy': 'Specific',
                'RobotIds': robot_ids or []
            }
        }

        response = requests.post(
            f'{self.base_url}/odata/Jobs/UiPath.Server.Configuration.OData.StartJobs',
            json=payload,
            headers={
                'Authorization': f'Bearer {self.token}',
                'X-UIPATH-OrganizationUnitId': str(folder_id),
                'Content-Type': 'application/json'
            }
        )
        response.raise_for_status()
        return response.json()
```

Notice that all the business logic—getting robots, starting jobs—is identical between Cloud and On-Premises. Only the initialization differs based on which URLs you provide.

Now, use environment variables for configuration:

```python
# Configuration
DEPLOYMENT = os.getenv('DEPLOYMENT_TYPE', 'cloud')

if DEPLOYMENT == 'cloud':
    client = OrchestratorClient('cloud',
        cloud_base_url=os.getenv('CLOUD_BASE_URL'),
        cloud_token_url=os.getenv('CLOUD_TOKEN_URL'),
        client_id=os.getenv('CLOUD_CLIENT_ID'),
        client_secret=os.getenv('CLOUD_CLIENT_SECRET')
    )
else:
    client = OrchestratorClient('onprem',
        onprem_base_url=os.getenv('ONPREM_BASE_URL'),
        onprem_token_url=os.getenv('ONPREM_TOKEN_URL'),
        client_id=os.getenv('ONPREM_CLIENT_ID'),
        client_secret=os.getenv('ONPREM_CLIENT_SECRET')
    )

# Use the client—same code regardless of deployment
robots = client.get_robots(folder_id=12345)
for robot in robots:
    print(f"{robot['Name']}: {robot['Type']}")
```

With this pattern, you can switch between Cloud and On-Premises just by changing environment variables. Your business logic stays the same. This makes your code portable and maintainable.

### Presenter Notes
- Walk through the code structure
- Emphasize that business logic is identical
- Point out the environment variable approach

---

## Section 12: Summary and Next Steps (2 minutes)

### Visual
- Summary slide with key points
- Comparison table: Cloud vs On-Premises
- Next module preview

### Script

Let's summarize what we've covered about Cloud versus On-Premises deployments.

Cloud is UiPath-hosted, with automatic updates, zero infrastructure management, and internet access required. It's ideal for quick deployment, companies without significant IT infrastructure, and organizations that want the latest features immediately. Most small to medium businesses choose Cloud.

On-Premises is self-hosted, giving you full control, the ability to deploy in air-gapped networks, and meeting strict compliance requirements. It requires significant IT resources to manage but is necessary for many regulated industries and large enterprises with specific security needs.

From an API perspective, the main differences are URL structure and authentication endpoints. Cloud includes organization and tenant in the URL path, On-Premises uses your custom domain. But the OAuth flow and API endpoints are largely identical, making it possible to write code that works with both.

We discussed hybrid deployments, which combine Cloud management with on-premises data processing. This is increasingly popular for organizations that want Cloud's ease of use but have data residency requirements.

When choosing between the two, consider: compliance requirements first—they often dictate the choice. Then IT capacity, time to value, scale, and feature needs.

As a developer, make sure your code is flexible. Use environment variables or configuration files to store URLs and credentials. Abstract the deployment type so your business logic works with both. Test against the specific versions you'll be deploying to, especially for On-Premises.

In the next module, we'll move from infrastructure concerns to actually working with automations. We'll cover Processes and Jobs—how to start jobs, monitor them, pass parameters, and schedule automation. This is where we start doing real automation work through the API.

Thanks for watching, and I'll see you in the next module!

### Presenter Notes
- Quick recap of key differences
- Emphasize practical takeaways for developers
- Build excitement for next module on actual automation

---

## Additional Teaching Notes

### Common Student Questions

**Q: Can I start with Cloud and migrate to On-Premises later?**
A: Yes, but it requires exporting processes, assets, and other resources and re-importing them in On-Premises. It's not automatic but is possible. Test the migration process in dev first.

**Q: Do robots always need to be where Orchestrator is?**
A: No! In hybrid deployments, robots can be on-premises while Orchestrator is in Cloud. Robots just need network access to Orchestrator.

**Q: What if I have multiple On-Premises Orchestrator instances?**
A: Treat each as a separate deployment. Each will have its own URL and credentials. Your code can manage multiple instances using the same patterns we showed.

**Q: How do licensing costs compare?**
A: Cloud is subscription per user/robot. On-Premises can be perpetual license or subscription, plus infrastructure costs. For small deployments, Cloud is usually cheaper. For 100+ robots, On-Premises can be more economical long-term.

**Q: Can I use Cloud for development and On-Premises for production?**
A: Yes, this is common. Develop and test in Cloud (quick and easy), deploy to On-Premises production (security and compliance). Just test thoroughly before production deployment.

### Demo Prerequisites
- Access to both Cloud and On-Premises Orchestrator (or screenshots/recordings)
- Credentials for both environments
- Python environment with requests library
- Example code that works with both

### Additional Examples
- Multi-environment setup (Cloud dev, On-Premises prod)
- Error handling for network issues specific to each model
- Monitoring and logging strategies for both
- Cost calculation examples

### Assessment Tips
- Quiz includes both conceptual and practical questions
- Emphasize decision-making criteria
- Test understanding of API endpoint differences
- Include hybrid deployment scenarios
