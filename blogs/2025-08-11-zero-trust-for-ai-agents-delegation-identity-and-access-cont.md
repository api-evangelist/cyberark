---
title: "Zero Trust for AI Agents: Delegation, Identity and Access Control"
url: "https://developer.cyberark.com/blog/zero-trust-for-ai-agents-delegation-identity-and-access-control/"
date: "Mon, 11 Aug 2025 10:59:18 +0000"
author: "Tomer Shtilman"
feed_url: "https://developer.cyberark.com/feed/"
---
<p>AI agents are no longer passive tools but are becoming trusted digital coworkers. They can fetch documents, schedule meetings, query APIs, and even make decisions on your behalf. But as they take on real responsibility, who are they acting as, and what happens when something goes wrong?</p>
<p>Unlike traditional software, these agents operate with a mix of autonomy and delegation. That makes identity and access control a security concern and a business-critical question. If your AI agent accesses private data or interacts with other systems, how do you ensure it’s doing so with the proper authority and nothing more?</p>
<p>In this post, we’ll unpack how defining, delegating, and enforcing AI agents&#8217; identities can protect your organization, enable safe collaboration, and unlock new capabilities without compromising control.</p>
<h3>The Risk of Delegation in AI Agents</h3>
<p>Delegation is powerful but dangerous if unchecked. We risk losing visibility and control when we grant AI agents the ability to act on our behalf. An agent with excessive or vague authority can accidentally leak data, trigger unintended actions, or become a vector for lateral movement in systems.</p>
<p>To delegate safely, we must bound what agents can do, on whose behalf, and under what conditions, just like we do with human users or service accounts. This means designing granular, auditable, revocable delegation forms that align with identity and intent.</p>
<p>A standard solution is delegation tokens, scoped credentials that let agents act on behalf of a user or another agent. This approach, formalized in <a href="https://www.rfc-editor.org/rfc/rfc8693.html" rel="noopener" target="_blank">RFC 8693</a>, enables layered identity and structured token exchange.</p>
<p>But issuing a token is just the beginning. The real challenge lies in the authorization decision: Who decides what the agent can do? Under what context? And how is that decision enforced and audited? Without strong, intentional authorization controls, delegation becomes a blind trust, opening the door to misuse, escalation, or unintended actions.</p>
<h3>Delegation via Token Exchange (RFC 8693)</h3>
<p>Before allowing one system or agent to act on behalf of another, you should consider how to represent both identities securely. Token exchange makes this possible by creating a new token that captures the relationship between the original subject and the acting party.</p>
<ul>
<li><strong>Subject Token</strong>: Represents the original identity (e.g., the user or client).</li>
<li><strong>Actor Token:</strong> Represents the acting party (typically a service, AI Agent, or application acting on behalf of the subject).</li>
</ul>
<p><em>The resulting delegation token includes both identities, allowing the agent (actor) to act on behalf of the user (subject).</em></p>
<h3>Imagine this:</h3>
<ul>
<li>Alice is a user.</li>
<li>An AI agent, let&#8217;s call it Agent A, wants to contact Service B on Alice’s behalf (for example, to book a flight for her).</li>
</ul>
<h3>Steps:</h3>
<ul>
<li>Alice authenticates, and her identity is turned into a subject token.</li>
<li>AI Agent A, acting on Alice’s behalf, has its actor token.</li>
<li>It requests a delegation token from the auth server by submitting both tokens.</li>
</ul>
<h2>Token Exchange Request</h2>
<pre class="prettyprint">POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded
grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&amp;subject_token=eyJhbGciOiJSUzI1NiIs... (Alice's token)
&amp;subject_token_type=urn:ietf:params:oauth:token-type:access_token
&amp;actor_token=eyJhbGciOiJSUzI1NiIs... (AI Agent A's token)
&amp;actor_token_type=urn:ietf:params:oauth:token-type:access_token
&amp;requested_token_type=urn:ietf:params:oauth:token-type:access_token
&amp;audience=https://service-b.example.com/

</pre>
<h2>Token Exchange Response</h2>
<pre class="prettyprint">{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "issued_token_type": "urn:ietf:params:oauth:token-type:access_token",
  "token_type": "Bearer",
  "expires_in": 3600
}</pre>
<p>The returned token is a delegation token, which embeds both subject and actor identities.</p>
<p><strong>e.g.,</strong></p>
<pre class="prettyprint">{ 

  "sub": "alice", 

  "act": { 

    "sub": "ai-agent-a" 

  } 

} 
</pre>
<p>This allows Service B to recognize that the request is being made by AI Agent A on behalf of Alice.</p>
<h2>What Happens During a Token Exchange</h2>
<p>The authorization server performs several checks and steps to issue a new token safely.</p>
<p><img alt="Authorization Server Performs " class="aligncenter wp-image-8238 size-full" height="2451" src="https://developer.cyberark.com/wp-content/uploads/2025/07/authorization-server-performs-v1-scaled.png" width="2560" /></p>
<h3>Constraints of Delegation Tokens and Permission Boundaries</h3>
<p>Although OAuth 2.0 token exchange supports robust delegation use cases, the delegation token produced does not automatically enforce or restrict the original authorization scope of the subject or the actor token.</p>
<p>It means that when you use OAuth 2.0 token exchange to let one party act on behalf of another, the new token you get (the delegation token) doesn’t automatically limit what that party can do based on the original permissions of either the user (subject) or the acting agent (actor). In other words, the delegation token might allow different or unintended access unless additional restrictions are implemented.</p>
<h2>Can we make educated or complex access decisions using the OAuth2 federation?</h2>
<p>When using OAuth2 federation, systems typically make simple, fixed decisions based on specific token details like a user ID or audience value. For example, AWS IAM’s OIDC federation grants access by matching static token claims to predefined roles without any advanced logic or analysis of the token’s content. This approach is intentional to keep the process straightforward and secure.</p>
<p>So, while OAuth2 federation enables identity delegation, it does not support making detailed or dynamic access decisions based on complete token information. AWS is a good example of simple, rigid federation mapping, focusing on exact matches rather than complex, context-based authorization.</p>
<h2>What privileges should a delegation token have?</h2>
<p>A simple way might be to give the delegation token only the permissions that the user (subject) and the AI or system (actor) have in common.</p>
<p>But this isn’t always the correct answer.</p>
<p>For example, the Australian government requires users to prove their age to access some social media sites, which can accidentally block AI agents from using those platforms. Instead of blocking AI completely, platforms could allow AI agents to act on behalf of verified users under strict rules. This means the AI represents a real user, can do tasks like checking friends or security settings, and must follow platform policies.</p>
<p>In this case, the delegation token shouldn’t just reflect the overlap of user and AI permissions; it needs to reflect controlled, trusted access for the AI acting for the user.</p>
<p style="text-align: center;"><strong>A delegation token may exceed the scope or resource access allowed initially to the subject or the actor individually.</strong></p>
<p>Delegation Token. If not carefully constrained, an actor (e.g., AI Agent A) could leverage its role to escalate privileges by requesting access that the subject (e.g., Alice) would not usually have or vice versa. Jailbreaking an AI agent is amplifying the threat caused by unconstrained delegation tokens.</p>
<p>Also, no automatic enforcement ensures that the delegation respects least privilege or original intent; it can potentially perform unintended or unpredictable operations or move laterally inside the target system.</p>
<h4>Policy Engine to Assist</h4>
<p>Open Policy Agent (OPA) is an open-source tool that helps organizations enforce fine-grained, flexible access control policies. Instead of hardcoding rules, OPA lets you write policies in a simple, declarative language and evaluate them anywhere in your app, API, or infrastructure. It makes dynamic, context-aware authorization decisions beyond basic static checks.</p>
<p>We integrate OPA to enforce fine-grained, dynamic authorization decisions on delegation tokens, representing the combined identity of a human user and an AI agent. When a delegation token includes the human subject (sub) and the acting AI agent (act), OPA can evaluate this context against real-time customizable policies.</p>
<p>Policies can specify what actions an AI agent can perform on behalf of specific users according to agent capabilities (e.g., booking a flight ), what scopes are permitted, and which resource access is valid under delegation. Using OPA, the system gains a flexible and auditable way to ensure that AI-driven actions stay within the intended boundaries of human consent and platform governance.</p>
<p><img alt="OPA Authorization Apply Request" class="aligncenter wp-image-8272 size-full" height="1216" src="https://developer.cyberark.com/wp-content/uploads/2025/07/ai-agent-identity-scaled.png" width="2560" /></p>
<h4>Example: OPA Authorization Apply Request</h4>
<pre class="prettyprint">{
  "input": {
    "actor": {
      "claims": {
        "Capabilities": "book_flight"
      }
    },
    "subject": {
      "claims": {
        "Department": "Operations",
        "Role": "Director"
      }
    }
  }
}

</pre>
<p><strong>Just-In-Time (JIT)</strong> Access means giving access to something only when it&#8217;s needed, and only for as long as necessary, <strong>no earlier, no longer</strong>.</p>
<p>In security (and often in OAuth or cloud systems), JIT access helps:</p>
<ul>
<li><strong>Reduce risk</strong> by minimizing how long someone (or something) holds sensitive permissions.</li>
<li><strong>Limit exposure</strong> if credentials are compromised.</li>
<li><strong>Enforce least privilege dynamically, not permanently</strong>.</li>
</ul>
<p><strong>Just Enough Access (JEA)</strong> means giving the minimum permissions necessary for someone (or something) to do their job    <strong>no more, no less.</strong></p>
<p>It&#8217;s about <strong>shrinking the blast radius </strong>if something goes wrong:</p>
<ul>
<li>If a person or AI agent only has &#8220;read&#8221; permission, they can&#8217;t accidentally (or maliciously) delete things.</li>
<li>If an AI agent only has access to a folder, it can&#8217;t snoop around the whole system.</li>
</ul>
<p>The response is dynamic, and the decision to assign a role to the target system is made in time. In this case, we choose a least-privilege role that only allows access to the needed resources. We then create a short-lived token that gives the AI agent Just In Time (JIT) access so it can perform just the required action based on what its token can do and what resources it can access (Just Enough Access—JEA).</p>
<h2>Wrap-up:</h2>
<p>Using this approach, we set clear and strict limits on what an AI agent can do. Instead of relying on simple fixed rules, we dynamically control the agent’s actions and access to resources based on the user’s original permissions combined with the agent’s roles, capabilities, and any other policies or context we define. This ensures safe, precise, and accountable delegation.</p>
<p>&nbsp;</p>
<p>The post <a href="https://developer.cyberark.com/blog/zero-trust-for-ai-agents-delegation-identity-and-access-control/">Zero Trust for AI Agents: Delegation, Identity and Access Control</a> appeared first on <a href="https://developer.cyberark.com">CyberArk Developer</a>.</p>
