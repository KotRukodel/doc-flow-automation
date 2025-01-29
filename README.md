# Documentation Processing Workflow with n8n

## Overview
This n8n workflow automates the technical documentation review and approval process. It integrates with GitHub and OpenAI to automatically process documentation updates, perform technical reviews, and manage the approval workflow.

## Features
- Automated triggering on GitHub repository changes
- AI-powered technical content review
- Automated issue creation for approved changes
- Documentation quality assurance workflow
- GitHub integration for version control

## Prerequisites
Before setting up this workflow, ensure you have:

1. n8n instance (version 1.0.0 or higher)
2. GitHub account with repository access
3. OpenAI API key
4. Repository with documentation in markdown format

## Installation

1. **Import the Workflow**
   - Open your n8n instance
   - Go to Workflows → Import from File
   - Upload the provided workflow JSON file
   - Save the workflow

2. **Configure Credentials**
   
   ### GitHub Configuration
   - Navigate to n8n Credentials
   - Add new GitHub API credentials
   - Configure:
     - Access Token
     - Repository permissions
   
   ### OpenAI Configuration
   - Add OpenAI API credentials in n8n Globals
   - Set the variable name as `OPENAI_API_KEY`
   - Input your OpenAI API key

3. **Configure Webhook**
   - In your GitHub repository:
     - Go to Settings → Webhooks
     - Add new webhook
     - Set payload URL to your n8n webhook URL
     - Select 'push' events
     - Enable the webhook

## Node Configuration

### 1. GitHub Trigger Node
- Monitors repository for push events
- Configuration:
  ```json
  {
    "authentication": "githubApi",
    "events": ["push"],
    "owner": "{{$node.Webhook.json.repository.owner.name}}",
    "repository": "{{$node.Webhook.json.repository.name}}"
  }
  ```

### 2. OpenAI Review Node
- Performs technical review of documentation changes
- Configuration:
  ```json
  {
    "method": "POST",
    "url": "https://api.openai.com/v1/completions",
    "authentication": "headerAuth",
    "bodyParameters": {
      "model": "gpt-4",
      "max_tokens": 500
    }
  }
  ```

### 3. Review Decision Node
- Evaluates AI review results
- Determines if documentation meets quality standards
- Configuration:
  ```json
  {
    "conditions": {
      "boolean": [
        {
          "value1": "={{$node.OpenAI Review.json.choices[0].text.includes('PASS')}}",
          "value2": true
        }
      ]
    }
  }
  ```

### 4. Create Issue Node
- Creates GitHub issues for approved documentation
- Configuration:
  ```json
  {
    "authentication": "githubApi",
    "owner": "={{$node.Github Trigger.json.repository.owner.name}}",
    "repository": "={{$node.Github Trigger.json.repository.name}}",
    "labels": ["documentation", "approved"]
  }
  ```

## Usage

1. **Activate the Workflow**
   - Click 'Active' toggle in n8n workflow editor
   - Verify webhook status in GitHub

2. **Testing the Workflow**
   - Make a documentation change in your repository
   - Commit and push the changes
   - Monitor the workflow execution in n8n
   - Check GitHub issues for review results

3. **Monitoring**
   - View execution history in n8n
   - Check workflow logs for errors
   - Monitor GitHub issues for approval status

## Customization

### Adjusting Review Criteria
Modify the OpenAI prompt in the HTTP Request node:
```json
{
  "prompt": "Review technical accuracy of:\n{{$node.Github Trigger.json.commits[0].message}}",
  "temperature": 0.7
}
```

### Adding Custom Labels
Modify the Create Issue node labels array:
```json
{
  "labels": ["documentation", "approved", "your-custom-label"]
}
```

## Troubleshooting

### Common Issues
1. **Webhook Not Triggering**
   - Verify webhook URL in GitHub
   - Check webhook logs in GitHub
   - Ensure n8n is accessible from GitHub

2. **OpenAI Review Failing**
   - Verify API key in globals
   - Check API rate limits
   - Verify prompt formatting

3. **GitHub Issues Not Creating**
   - Check GitHub API credentials
   - Verify repository permissions
   - Check issue creation permissions

## Best Practices

1. **Documentation Standards**
   - Use consistent commit message format
   - Include relevant documentation context
   - Follow repository documentation guidelines

2. **Workflow Management**
   - Regularly monitor execution logs
   - Update API keys before expiration
   - Maintain webhook security

3. **Version Control**
   - Keep workflow JSON backup
   - Document workflow modifications
   - Track configuration changes

## Support
For issues and support:
1. Check n8n documentation: https://docs.n8n.io/
2. Review GitHub API documentation
3. Consult OpenAI API documentation

## Contributing
To contribute to this workflow:
1. Fork the repository
2. Make your changes
3. Submit a pull request with detailed description
4. Include testing results

## License
This workflow is released under the MIT License. See LICENSE file for details.
