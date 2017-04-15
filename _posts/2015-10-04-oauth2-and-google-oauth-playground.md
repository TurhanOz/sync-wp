---
ID: 341
post_title: OAuth2 and Google OAuth Playground
author: turhan
post_date: 2015-10-04 14:57:37
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/oauth2-and-google-oauth-playground/
published: true
---
<p class="p1"><span class="s1">If you are a developer, chances are that you are using APIs that requires authorization to be used. That is especially the case with most of the Google APIs. So, you should at least be familiar with OAuth2.</span></p>
<p class="p1"><span class="s1">The aim of this article is not to describe what OAuth2 is (there are dozens of literature on it), but simply a reminder of the standard, and pointers on how to use Google OAuth2 Playground which is a super easy interactive demonstration tool using Google APIs.</span></p>

<h1 class="p1"><span class="s1">OAuth 2, reminder :</span></h1>
<h2 class="p1"><span class="s1">Roles</span></h2>
<p class="p1"><span class="s1">OAuth2 is not an authorization protocol, it’s a secure delegated access protocol. In this manner, you have to consider 4 distincts roles within this process : </span></p>

<ul class="ul1">
	<li class="li1"><span class="s1">the ressource owner (typically you, or me or anyone…)</span></li>
	<li class="li1"><span class="s1">the ressource server where the owner stores his data (for instance entries in Google Calendar)</span></li>
	<li class="li1"><span class="s1">the client application (webpage, android app…) that will need to access the ressource of the owner (to display the Calendar entries for instance)</span></li>
	<li class="li1"><span class="s1">the authorization Server that grants access to the application (based on the owner approval).</span></li>
</ul>
<h2 class="p1"><span class="s1">Token</span></h2>
Whenever the client application needs to access a ressource, it first asks to the authorization server a Token (access or refresh token). This token is then used to access to the ressource located on the ressource server (the token could be passed in various way, but the most common is through the header of the request).
<h2 class="p1"><span class="s1">Scope</span></h2>
<p class="p1"><span class="s1">Token are used in a defined scope, which is a spectrum of ressources accessible with that token.</span></p>

<h1 class="p1"><span class="s1">Google OAuth Playground</span></h1>
<p class="p1"><span class="s1"><a href="https://developers.google.com/oauthplayground/">Google OAuth Playground</a> is a sandbox where you can play with OAuth2 and the APIs that support it. </span><span class="s1">For our use case, we will test the <a href="https://developers.google.com/beacons/proximity/guides">Proximity Beacon API</a> which requires OAuth2.</span></p>
<p class="p1"><span class="s1">There is already a description of the process for <a href="https://developers.google.com/beacons/proximity/how-tos/authorizing">authorizing and Authenticating Proximity Beacon API requests</a>, </span><span class="s1">but we’ll cover that with some screenshots.</span></p>

<h2 class="p1">Credentials</h2>
<p class="p1"><span class="s1">As described on the <a href="https://developers.google.com/beacons/proximity/how-tos/authorizing">documentation</a>, the first thing you need to do is to go to the <a href="https://console.developers.google.com/">Google Developers Console</a>, create a project, activate the Google Beacon API, then go to the Credentials Section in order to get a client ID for a web applications (that we'll use for the Playground)</span></p>
<p class="p2"><a href="http://turhanoz.com/wp-content/uploads/2015/10/1.Credential.png"><img class="alignnone wp-image-346 size-large" src="http://turhanoz.com/wp-content/uploads/2015/10/1.Credential-1024x537.png" alt="1.Credential" width="634" height="332" /></a></p>
<p class="p2"><a href="http://turhanoz.com/wp-content/uploads/2015/10/2.Credentials.png"><img class="alignnone wp-image-347 size-large" src="http://turhanoz.com/wp-content/uploads/2015/10/2.Credentials-1024x668.png" alt="2.Credentials" width="634" height="414" /></a></p>
<p class="p2"><span class="s2">Be carefull to set </span><span class="s1"><em>https://developers.google.com/oauthplayground</em> as a redirect URI</span></p>

<h2 class="p2">PlayGround</h2>
<p class="p1"><span class="s1">Go to the the <a href="https://developers.google.com/oauthplayground">OAuth 2.0 Playground</a></span><span class="s2"> and configure the OAuth Credentials (generated previously from developer console. Copy paste it there, as shown below)</span></p>
<p class="p1"><a href="http://turhanoz.com/wp-content/uploads/2015/10/3.OAuthConf.png"><img class="alignnone wp-image-348 size-large" src="http://turhanoz.com/wp-content/uploads/2015/10/3.OAuthConf-1024x997.png" alt="3.OAuthConf" width="634" height="617" /></a></p>
<p class="p1"><span class="s1">On the left side menu of the Playground, you have to set the scope for the Proximity Beacon API </span><span class="s2">into the field labeled <em><b>Input your own scopes</b></em>: </span><em><span class="s3">https://www.googleapis.com/auth/userlocation.beacon.registry</span></em></p>
<p class="p1"><a href="http://turhanoz.com/wp-content/uploads/2015/10/4.-Scope.png"><img class="alignnone wp-image-349 size-large" src="http://turhanoz.com/wp-content/uploads/2015/10/4.-Scope-927x1024.png" alt="4. Scope" width="634" height="700" /></a></p>
<p class="p1"><span class="s1">As a result, the token will be generated and automatically used for next steps.</span></p>
<p class="p1"><a href="http://turhanoz.com/wp-content/uploads/2015/10/5.Token_.png"><img class="alignnone wp-image-350 " src="http://turhanoz.com/wp-content/uploads/2015/10/5.Token_.png" alt="5.Token" width="617" height="423" /></a></p>
<p class="p1"><span class="s1">Now, all the magic comes, you can request your API that request authorization (for instance <em>https://proximitybeacon.googleapis.com/v1beta1/beacons</em>)</span></p>
<p class="p1"><a href="http://turhanoz.com/wp-content/uploads/2015/10/6.Request.png"><img class="alignnone wp-image-351 " src="http://turhanoz.com/wp-content/uploads/2015/10/6.Request.png" alt="6.Request" width="610" height="620" /></a></p>
<p class="p1"><span class="s1">As you can see, you can add headers, add request body and so… </span><span class="s1">In a word, you can easily test the API you are interested in and see the full result (header &amp; body) :)</span></p>
<p class="p1">You'll get the response of your request which requires authorization :)</p>
<p class="p1"><a href="http://turhanoz.com/wp-content/uploads/2015/10/7.response.png"><img class="alignnone wp-image-355" src="http://turhanoz.com/wp-content/uploads/2015/10/7.response-294x300.png" alt="7.response" width="428" height="437" /></a></p>
<p class="p1">Do not hesitate to over use this playground, it should definitely be used as part of API exploration (more than reading static documentation :)</p>