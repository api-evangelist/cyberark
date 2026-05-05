---
title: "From Claude Code Scan to Automated Secret Remediation: Building a Secure MCP Server for AI Agents"
url: "https://developer.cyberark.com/blog/from-claude-code-scan-to-automated-secret-remediation-building-a-secure-mcp-server-for-ai-agents/"
date: "Fri, 06 Mar 2026 20:47:12 +0000"
author: "Or Geisler"
feed_url: "https://developer.cyberark.com/feed/"
---
<p>Hardcoded secrets remain one of the most persistent security failures in modern software development. API keys committed to Git, tokens embedded in configuration files, credentials copied between environments. Everyone knows this is risky, yet it keeps happening because the alternatives are slow, manual, and disrupt developer flow.</p>
<p>Over the past months, I built an MCP server integration for CyberArk Secrets Manager SaaS that connects AI agents, IDE tooling, identity, and secret management into a single automated remediation workflow.</p>
<p>Recently, while experimenting with the new security capabilities in Claude Code, I started using it as the detection layer for a workflow I had been building, an MCP server integration for CyberArk Secrets Manager SaaS. Instead of treating security scanning and remediation as separate steps, the agent identifies hardcoded secrets and invokes MCP tools that replace them with managed secrets, and refactor code safely.</p>
<p>The result is a fully automated remediation loop that runs inside the developer workflow.</p>
<p>This post walks through the engineering journey from an early proof of concept to a beta-ready MCP server, the architecture behind it, and why OAuth with PKCE became a foundational security decision.</p>
<p><img alt="CyberArk Secrets Manager " class="aligncenter size-full wp-image-8419" height="411" src="https://developer.cyberark.com/wp-content/uploads/2026/03/cyberark-secrets-manager.png" width="960" /></p>
<h2>Hardcoded Secrets in Real Time</h2>
<p>Before diving into architecture, here’s a short end-to-end demo of the system in action.</p>
<p>In this flow:</p>
<ol>
<li>Claude Code scans a repository and detects exposed AWS and MongoDB credentials.</li>
<li>The agent calls MCP tools to authenticate through Identity.</li>
<li>Secrets are created automatically in Secrets Manager SaaS.</li>
<li>A workload identity is provisioned with least-privilege access.</li>
<li>The code is refactored to retrieve secrets securely instead of embedding credentials.</li>
</ol>
<p></p>
<div class="wistia_responsive_padding" style="padding: 57.92% 0 0 0;">
<div class="wistia_responsive_wrapper" style="height: 100%; width: 100%;">
<div class="wistia_embed wistia_async_6nw83l1eeq seo=false videoFoam=true" style="height: 100%; width: 100%;">
<div class="wistia_swatch" style="height: 100%; overflow: hidden; width: 100%;"><img alt="" src="https://fast.wistia.com/embed/medias/6nw83l1eeq/swatch" /></div>
</div>
</div>
</div>
<p>&nbsp;</p>
<h2>The Original Problem: Hardcoded Secrets at Scale</h2>
<p>The story started with a security champion reviewing multiple repositories and finding the same pattern everywhere:</p>
<ul>
<li>API keys embedded in code</li>
<li>Tokens committed to Git</li>
<li>Environment files checked in</li>
</ul>
<p>The directive was clear: <strong>All static secrets must be removed from the codebase.</strong></p>
<p>Manually fixing this across dozens of repositories is slow, error-prone, and usually incomplete. Even worse, security validation often occurs late, after the code has already been merged or deployed.</p>
<p>We want a system that works where developers already are:</p>
<ul>
<li>Inside IDEs</li>
<li>During code generation</li>
<li>During refactoring</li>
<li>Before the code is merged</li>
</ul>
<p>And it had to work with AI agents.</p>
<h2>Architecture Overview</h2>
<p>At a high level, the flow looks like this:</p>
<ul>
<li>An AI agent or any other detection tool detects a hardcoded secret</li>
<li>MCP server authenticates the user via CyberArk Identity (OAuth)</li>
<li>MCP server creates branches, secrets, and workloads in CyberArk’s Secrets Manager SaaS</li>
<li>The agent refactors the code to fetch secrets securely</li>
</ul>
<p>Key components:</p>
<ul>
<li>Claude Code acting as the detection layer</li>
<li>MCP server running in Docker</li>
<li>CyberArk Identity</li>
<li>Secrets Manager SaaS (formerly Conjur Cloud)</li>
</ul>
<p>The MCP server acts as a bridge to Secrets Manager SaaS. It never stores secrets, never embeds credentials, and only operates with short-lived tokens.</p>
<h3>High-Level Sequence Flow</h3>
<p>The following diagram shows the full system context and trust boundaries. It highlights where identity, secrets, and AI tooling intersect, and why the MCP server is the enforcement point.</p>
<p><img alt="Architecture Overview Diagram " class="aligncenter size-full wp-image-8427" height="614" src="https://developer.cyberark.com/wp-content/uploads/2026/03/architecture-overview-diagram.png" width="1902" /></p>
<p>Key things to notice:</p>
<ul>
<li>The MCP server runs inside the customer environment, not in the SaaS control plane</li>
<li>Authentication always flows through CyberArk Identity</li>
<li>Secrets Manager SaaS is only accessed with short-lived, user-scoped tokens</li>
<li>AI agents never receive secret values</li>
</ul>
<h2>End-to-End Remediation Flow</h2>
<p>A full remediation session looks like this:</p>
<p><strong>1. Scan</strong><br />
The process begins with Claude Code running a security scan inside the developer workflow. Instead of creating a custom detection engine, the MCP server consumes the findings produced by the agent and translates them into controlled remediation actions.</p>
<p><img alt="Claude Code identifying hardcoded AWS and MongoDB credentials before remediation begins." class="aligncenter size-full wp-image-8435" height="471" src="https://developer.cyberark.com/wp-content/uploads/2026/03/claude-code-identifying-hardcoded.png" width="1802" /></p>
<p><strong>2. Authenticate</strong><br />
The AI agent connects to the MCP server. If no valid token exists, a browser login is triggered via CyberArk Identity.</p>
<p><strong>3. Create branch</strong><br />
A branch is created to isolate changes safely.</p>
<p><strong>4. Create secret</strong><br />
For each finding, the MCP server creates a managed secret in Secrets Manager SaaS.</p>
<p><strong>5. Create workload</strong><br />
A workload identity with read-only permissions for the secret is created for the service.</p>
<p><strong>6. Refactor code</strong><br />
The AI agent replaces the hardcoded value with an SDK-based retrieval call.</p>
<p><strong>7. Review and merge</strong><br />
The user reviews the diff and merges a fully remediated, secret-free commit.</p>
<p>This entire loop runs within the developer workflow and takes a few minutes rather than days.</p>
<h2>Authentication Design: Why PKCE Instead of Client Credentials</h2>
<p>One of the earliest design decisions was how the MCP server should authenticate.</p>
<p>The naive approach is OAuth client credentials:</p>
<ul>
<li>Store CLIENT_ID and CLIENT_SECRET on the Client side</li>
<li>Exchange them for tokens</li>
<li>Use tokens to call Identity and Secrets Manager SaaS</li>
</ul>
<p>We rejected this early for several reasons:</p>
<ul>
<li>Client secrets become long-lived, high-value credentials</li>
<li>Any compromise of the Client secret may lead to tenant access</li>
<li>Secrets would need rotation, storage, and protection</li>
</ul>
<p>Instead, we chose OAuth Authorization Code Flow with PKCE.</p>
<h2>What PKCE Actually Solves</h2>
<p>PKCE (Proof Key for Code Exchange) was designed to protect public clients that cannot safely store secrets, such as browsers, mobile apps, and in our case, local MCP servers and developer tooling.</p>
<p>The flow works like this:</p>
<p><img alt="MCP Server -&gt; Generate code_verifier derive code_challenge -&gt; CyberArk Identity -&gt; User login authorization_code -&gt; MCP Server -&gt; access_token" class="aligncenter wp-image-8443 size-full" height="332" src="https://developer.cyberark.com/wp-content/uploads/2026/03/generate-code-verifier.png" width="1498" /></p>
<p>The critical property:</p>
<ul>
<li>No client secret is ever stored</li>
<li>The authorization code is useless without the original verifier</li>
<li>Token theft via interception becomes ineffective</li>
</ul>
<p>Even if an attacker captures the authorization code, they cannot exchange it without the verifier that only exists in memory on the MCP server.</p>
<h2>Why This Matters in an AI Tooling Context</h2>
<p>Our MCP server runs:</p>
<ul>
<li>Locally</li>
<li>In ephemeral Docker containers</li>
</ul>
<p>Storing long-lived client secrets in these environments is fundamentally unsafe.</p>
<p>OAuth2 + PKCE in Identity authentication allows:</p>
<ul>
<li>No static credentials on the client side</li>
<li>Tokens are short-lived</li>
<li>Authentication is bound to an actual human login</li>
<li>Revocation and audit are centralized in Identity</li>
<li>Full audit trail</li>
</ul>
<h2>Why Not Use Environment Variables?</h2>
<p>A common question we got early was: “Why not just inject secrets as environment variables and be done with it?”</p>
<p>Environment variables are simple, but they break down quickly at scale.</p>
<h3>Environment variables do not solve the distribution</h3>
<ul>
<li>Environment variables leak more than people expect. They often end up in crash dumps and debug logs or are accidentally printed</li>
<li>Environment variables are not identity-aware, and secret access is not audited.</li>
<li>Environment variables cannot be rotated.</li>
</ul>
<h2>Security Principals Applied</h2>
<p>Several principles guided the design.</p>
<h3>Least privilege</h3>
<ul>
<li>Each workload only receives access to the specific secrets it needs</li>
<li>No wildcard policies</li>
<li>No shared identities</li>
</ul>
<h3>Human confirmation</h3>
<p>Before creating or auto-approving:</p>
<ul>
<li>Path is confirmed</li>
<li>Secret creation can be approved or rejected</li>
<li>Code changes are reviewed before merging</li>
</ul>
<p>This prevents silent mass changes and keeps developers in control.</p>
<h3>No secret exposure</h3>
<ul>
<li>Secrets are never printed</li>
<li>Never logged</li>
<li>Never returned to the AI agent</li>
</ul>
<p><img alt="Agent generated remediation summary showing secrets created, workloads provisioned, and code changes applied through MCP" class="aligncenter size-full " height="774" src="https://developer.cyberark.com/wp-content/uploads/2026/03/agent-generated-remediation.png" style="width: 840px;" width="843" /></p>
<h2>Lessons Learned</h2>
<p>Building an AI-native secret remediation workflow surfaced several non-obvious lessons.</p>
<h3>Authentication design matters more than API design</h3>
<p>The decision to use OAuth with PKCE removed an entire class of long-lived credential risks before any policy or validation logic was added. Security posture improved with the construction.</p>
<h3>Automation must remain explainable</h3>
<p>Every action taken by the MCP server is visible and reviewable:</p>
<ul>
<li>Which secret was created</li>
<li>Where it was stored</li>
<li>Which workload can access it</li>
<li>What code was modified</li>
</ul>
<p>Developers trust systems they can reason about and audit.</p>
<h3>MCP is a natural security control point</h3>
<p>Because MCP sits between AI agents, developer tooling, identity, and Secrets Manager SaaS, it becomes the correct place to enforce policy, least privilege, and human confirmation.</p>
<h2>Closing Thoughts</h2>
<p>Hardcoded secrets are not a tooling failure. They are a workflow failure.</p>
<p>By embedding Secrets Manager SaaS directly into AI-assisted development workflows, it becomes possible to:</p>
<ul>
<li>Eliminate static credentials</li>
<li>Reduce remediation time from days to minutes</li>
<li>Improve auditability and compliance</li>
<li>Keep developers in flow</li>
</ul>
<p>The MCP server is not simply a connector. It is an AI-native security enforcement layer.</p>
<p>As AI agents become first-class participants in software development, security controls must evolve with them. Identity-aware MCP integrations are one step in that direction.</p>
<p>Want to wire your AI tools directly to Secrets Manager, SaaS? The MCP Server handles the OAuth handshake, scopes permissions to least privilege, and swaps hard-coded secrets for variable references. It&#8217;s a Docker container, it&#8217;s free, and it&#8217;s sitting on the Marketplace right now.</p>
<p><a href="https://community.cyberark.com/marketplace/s/#software-aK4Vy00000001NVKAY-" rel="noopener" target="_blank">Grab the beta and try it</a></p>
<p>The post <a href="https://developer.cyberark.com/blog/from-claude-code-scan-to-automated-secret-remediation-building-a-secure-mcp-server-for-ai-agents/">From Claude Code Scan to Automated Secret Remediation: Building a Secure MCP Server for AI Agents</a> appeared first on <a href="https://developer.cyberark.com">CyberArk Developer</a>.</p>
