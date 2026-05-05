---
title: "Introducing the Conjur Bitbucket Pipe"
url: "https://developer.cyberark.com/blog/introducing-the-conjur-bitbucket-pipe/"
date: "Fri, 04 Jul 2025 02:58:02 +0000"
author: "Shlomo Heigh"
feed_url: "https://developer.cyberark.com/feed/"
---
<h2>Introduction</h2>
<p>At CyberArk, we&#8217;re always trying to find ways to make it easier for developers to securely manage secrets wherever their code runs. That&#8217;s why we&#8217;re excited to introduce the <a href="https://github.com/cyberark/conjur-bitbucket-pipe" rel="noopener" target="_blank">Conjur Bitbucket Pipe!</a> This integration allows CI pipelines that run in Bitbucket Cloud to easily retrieve secrets from Conjur for use in their builds and deployments.</p>
<h2>What is the Conjur Bitbucket Pipe?</h2>
<p>The Conjur Bitbucket Pipe is a new integration that allows you to connect your Bitbucket Cloud pipelines to Conjur. It works by authenticating with Conjur using the Bitbucket secure OIDC token, which means there&#8217;s no need to store any Conjur credentials in your Bitbucket repository. This reduces the risk of credential exposure and simplifies secret management, allowing teams to focus on delivering code without worrying about managing or rotating sensitive credentials. The integration provides a simple way to retrieve secret values stored in Conjur OSS, Conjur Enterprise, or Conjur Cloud so they can be used by your CI/CD pipelines.</p>
<h2>How can I use it?</h2>
<h3>Pre-requisites</h3>
<p>To use the Conjur Bitbucket Pipe, you need to have:</p>
<ul>
<li>A <a href="https://bitbucket.org/" rel="noopener" target="_blank">Bitbucket Cloud </a>account</li>
<li>A Bitbucket repository with a <a href="https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/" rel="noopener" target="_blank">pipeline</a> configured</li>
<li>A <a href="https://www.conjur.org/" rel="noopener" target="_blank">Conjur OSS</a>, <a href="https://www.cyberark.com/products/secrets-manager-self-hosted/">Conjur Enterprise</a>, or <a href="https://www.cyberark.com/products/secrets-manager-saas/">Conjur Cloud</a> account</li>
</ul>
<h2>Configure Authentication</h2>
<p>Before you can use the pipe, you need to configure authentication between Bitbucket and Conjur. This is done by creating a Conjur policy that allows the Bitbucket pipeline to authenticate using the OIDC token. Here&#8217;s an example:</p>
<pre class="prettyprint"># Create the authenticator 

- !policy 

  id: conjur/authn-jwt/bitbucket 

  body: 

    - !webservice 

    - !variable provider-uri 

    - !variable token-app-property 

    - !variable identity-path 
   
    - !group authenticatable 

    - !permit 

      role: !group authenticatable 

      privilege: [ read, authenticate ] 

      resource: !webservice 

# Create hosts for your Bitbucket pipelines 

- !policy 

  id: bitbucket-pipelines 

  body: 

    - !group 

    - &amp;hosts 

      - !host 

# Replace this with your repositoryUuid. You must include the curly braces and double quotes! 

        id: "{&lt;bitbucket-repository-uuid&gt;}"

        annotations: 

          authn-jwt/bitbucket/repositoryUuid: "{&lt;bitbucket-repository-uuid&gt;}" # Replace this as well 

# Add more hosts here for other Bitbucket repositories if needed 
  
     - !grant 

      role: !group 

      members: *hosts 

# Create some secrets for the pipelines to use 

    - &amp;variables 

      - !variable username 

      - !variable password 
  
# Allow the pipelines to read the variables 

    - !permit 

      role: !group 

      privilege: [ read, execute ] 

      resource: *variables 

# Add the pipelines to the group that can authenticate using authn-jwt/bitbucket 

- !grant 

  role: !group authn-jwt/bitbucket/authenticatable 

  members: !group bitbucket-pipelines 
</pre>
<p>After loading this policy into Conjur, add values for the authenticator variables:</p>
<pre class="prettyprint"># Replace `&lt;workspace-name&gt;` with your Bitbucket workspace name
 conjur variable set -i conjur/authn-jwt/bitbucket/provider-uri -v "https://api.bitbucket.org/2.0/workspaces/&lt;workspace-name&gt;/pipelines-config/identity/oidc"</pre>
<pre class="prettyprint">conjur variable set -i conjur/authn-jwt/bitbucket/token-app-property -v "repositoryUuid" 

# This is the path in the Conjur policy where the Bitbucket pipeline hosts are defined 

conjur variable set -i conjur/authn-jwt/bitbucket/identity-path -v "bitbucket-pipelines" 
</pre>
<p>Now Conjur is ready to authenticate your Bitbucket pipelines!</p>
<h2>Using the Pipe in Your Pipeline</h2>
<pre class="prettyprint">To use the Conjur Bitbucket Pipe in your pipeline, you can add it to your bitbucket-pipelines.yml file like this: 
- step: 
  name: 'Retrieve secrets from Conjur' 
  oidc: true # This instructs Bitbucket to use provide OIDC credentials to the Pipe 

  script: 

    - pipe: cyberark-conjur/conjur-bitbucket-pipe:0.0.8 

      variables: 

        CONJUR_URL: 'https://&lt;your-conjur-url&gt;' 

        CONJUR_ACCOUNT: '&lt;your-conjur-account&gt;' # Defaults to 'conjur' 

        CONJUR_SERVICE_ID: 'bitbucket' # Service ID of the JWT Authenticator in Conjur. Defaults to 'bitbucket' 

        SECRETS: 'bitbucket-pipelines/username,bitbucket-pipelines/password' # Comma-separated list of Conjur variable IDs 

    - . ./.secrets/load_secrets.sh # This command loads the secrets into environment variables 

 # Now you can access the secrets as environment variables 
 # For example, 

    - curl -u $username:$password https://some-api.example.com/resource
</pre>
<p>You can see the full documentation, including advanced usage options, on our <a href="https://github.com/cyberark/conjur-bitbucket-pipe" rel="noopener" target="_blank">GitHub repository</a>.</p>
<h2>Conclusion</h2>
<p>The Conjur Bitbucket Pipe makes it easy to securely manage secrets in your Bitbucket Cloud pipelines. By using OIDC authentication, you can avoid storing Conjur credentials in your repository, and you can easily retrieve secrets from Conjur for use in your builds and deployments. We hope this integration helps you streamline your CI/CD workflows while keeping your secrets secure. If you have any questions or feedback, please feel free to reach out to us on our <a href="https://github.com/cyberark/conjur-bitbucket-pipe" rel="noopener" target="_blank">GitHub repository</a> or <a href="https://community.cyberark.com/" rel="noopener" target="_blank">CyberArk Community</a>. Happy coding!</p>
<p>The post <a href="https://developer.cyberark.com/blog/introducing-the-conjur-bitbucket-pipe/">Introducing the Conjur Bitbucket Pipe</a> appeared first on <a href="https://developer.cyberark.com">CyberArk Developer</a>.</p>
