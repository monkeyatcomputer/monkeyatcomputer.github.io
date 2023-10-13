---
permalink: /presence-using-the-unified-communications-web-api
title: "Presence using the Unified Communications Web API (UCWA) "
description: "More JSON than you can poke 6 authentication steps at"
image:
  path: "/assets/img/presence.jpg"
---

## Introduction

Since Lync Server 2013 Cumulative Update 1 (February 2013), it has been possible to use the Unified Communications Web API (UCWA) to provide web-based communications interactions with Lync users. Microsoft provide a set of JavaScript helper libraries but unfortunately they have forgotten about those of us that have other non-web based ideas.

This is my second attempt at working with UCWA and now the server side has been updated to Skype for Business.

Before we can get presence from the Skype for Business (Lync) server, we need to create an application. Doing so requires five API calls to discover, authenticate and create an application. Once we have our application we can make it "available"  with another API call.

![UCWA Authentication Workflow](/assets/img/ic673053.png)
_from [https://msdn.microsoft.com/en-us/library/office/dn356799.aspx](https://msdn.microsoft.com/en-us/library/office/dn356799.aspx "Create an application")_

The steps are:

1.  Send a GET request on the Autodiscovery URL.
2.  Send a GET request on the user URL, indicating that we want to authenticate as a user.
3.  Send a POST request on the OAuth URL.
4.  Send another GET request on the user resource, passing the OAuth token in the Authorization header of the request.
5.  Send a POST request on the applications resource.
6.  Send a GET request to the "makeMeAvailable" URL.

I'll be covering steps 2 to 6 as there is nothing special about the autodiscover URL.

## Initialisation Code

I'm using Caliburn.Micro and Autofac in my app meaning the constructor contains some IoC magic. The program flow is largely copied/ported from the Microsoft JavaScript helpers using the same state machine pattern.

```csharp
using Caliburn.Micro;
using IotPresence.Models.Ucwa;
using IotPresence.Settings;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Text.RegularExpressions;
using System.Threading.Tasks;

internal class AuthenticationService : IAuthenticationService
{
	private readonly IEventAggregator eventAggregator;
	private readonly ISettingsRepository settingsRepository;
	private readonly IApplicationCache cache;

	private HttpClient httpClient;
	private int currentState = 0;
	private bool authenticated = false;
	private int authErrorCounter = 0;

	public AuthenticationService(IEventAggregator eventAggregator, ISettingsRepository settingsRepository, IApplicationCache cache)
	{
		this.eventAggregator = eventAggregator;
		this.settingsRepository = settingsRepository;
		this.cache = cache;

		this.httpClient = new HttpClient();
		httpClient.DefaultRequestHeaders.Accept.Clear();
		httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

		this.eventAggregator.Subscribe(this); // We need to publish Caliburn.Micro events
	}

	public async Task Start()
	{
		if (!String.IsNullOrEmpty(cache.UserUrl))
			await HandleState(new StateData(cache.UserUrl)).ConfigureAwait(false);
		else
			throw new InvalidOperationException("Unable to start authentication until auto discover process complete");
	}
}
```

*   Prepare HttpClient for use. We're dealing with JSON here but we could just as easily set the MIME type for XML.
*   Start our recursive state machine or throw an error if we haven't been given a web service URL.

## State Machine

### HandleState()

```csharp
private async Task HandleState(StateData data)
{
	bool success = ProcessStateData(data);

	if (success)
	{
		switch (currentState)
		{
			case 0:
				// Start Authentication
				await StartAuthentication(data);
				break;
			case 1:
				// Handle Authorization
				await HandleAuthorization(data);
				break;
			case 2:
				// Authenticate
				await Authenticate(data);
				break;
			case 3:
				// Create Application
				await CreateApplication(data);
				break;
			case 4:
				// Make me available
				await MakeMeAvailable(data);
				break;
			case 5:
				authenticated = true;
				break;
			default:
				break;
		}
	}
}
```

*   Pass our state to the ProcessStateData method and if everything is as we expect, we can call our handler methods
*   Note that I am not yet handling redirects and other edge cases like the Microsoft UCWA JavaScript helpers.

### ProcessStateData()

```csharp
private bool ProcessStateData(StateData data)
{
	if (data != null)
	{
		if (HandleRedirect(data))
			return false;

		if (data.StatusCode != null)
		{ 
			switch (data.StatusCode)
			{
				case HttpStatusCode.OK:
				case HttpStatusCode.Created:
				case HttpStatusCode.NoContent:
					// Intentional fall-through for all expected 2xx states
					currentState++;

					if (currentState == 3
						&& data.ApplicationsUrl == null)
						currentState = 3;

					break;
				case HttpStatusCode.Unauthorized:
					// 401 means it's time to supply credentials
					data.OAuthUrl = GetOAuthUrl(data.WwwAuthenticate);

					// Track how many failed authorize attempts occur
					if (currentState == 1)
						authErrorCounter++;
					else
						currentState++;
					break;
				case HttpStatusCode.BadRequest:
				case HttpStatusCode.NotFound:
					// Reset for either 400 or 404
					ResetState();
					return false;
				default:
					ResetState();
					return false;
			}
		}

		return true;
	}

	ResetState();
	return false;
}
```

*   When the response from our web service call is a 2xx status code, we increment our state and move to the next step.
*   When we get challenged for authentication, we also move to the next step but keep a count of how many times we have been challenged. Not yet handled properly.

## Authenticate all the (Internet of) Things

### StartAuthentication()

```csharp
private async Task StartAuthentication(StateData data)
{
	var request = new HttpRequestMessage()
	{
		RequestUri = new Uri(data.UserUrl),
		Method = HttpMethod.Get
	};

	var response = await httpClient.SendAsync(request);

	data.StatusCode = response.StatusCode;
	data.WwwAuthenticate = response.Headers.WwwAuthenticate.ToString();

	await HandleState(data);
}
```

*   Make a request to the user URL, obtained using lyncdiscover, and get the authentication headers as this is where our OAuth URL is obtained.

### HandleAuthorization()

```csharp
private async Task HandleAuthorization(StateData data)
{
	var settings = settingsRepository.Read();

	var postData = new List>();
	postData.Add(new KeyValuePair("grant_type", "password"));
	postData.Add(new KeyValuePair("username", settings.UserName));
	postData.Add(new KeyValuePair("password", settings.Password));

	var request = new HttpRequestMessage()
	{
		RequestUri = new Uri(data.OAuthUrl),
		Method = HttpMethod.Post,
		Content = new FormUrlEncodedContent(postData)
	};

	var response = await httpClient.SendAsync(request);

	data.StatusCode = response.StatusCode;

	if (response.StatusCode == HttpStatusCode.OK)
	{
		var content = await response.Content.ReadAsStringAsync();
		var parsed = JObject.Parse(content);

		if (!this.authenticated)
		{
			data.UserToken = data.ApplicationToken = (string)(parsed["access_token"] as JValue).Value;
			data.UserTokenType = data.ApplicationTokenType = (string)(parsed["token_type"] as JValue).Value;
		}
		else
		{
			data.ApplicationToken = (string)(parsed["access_token"] as JValue).Value;
			data.ApplicationTokenType = (string)(parsed["token_type"] as JValue).Value;
		}

		// we need this externally
		this.cache.ApplicationToken = data.ApplicationToken;
		this.cache.ApplicationTokenType = data.ApplicationTokenType;
	}

	await HandleState(data);
}
```

*   Prepare authentication data to be posted. We're using password authentication as I cannot see any of the other methods working from Windows 10 IoT.
*   Post data to the OAuth URL
*   Retrieve OAuth token for the user and application from JSON response.
*   Now that we have our OAuth token, we can  authenticate with UCWA in the next step.

### Authenticate()

```csharp
private async Task Authenticate(StateData data)
{
	var request = new HttpRequestMessage()
	{
		RequestUri = new Uri(data.UserUrl),
		Method = HttpMethod.Get
	};

	request.Headers.Authorization = new AuthenticationHeaderValue(data.UserTokenType, data.UserToken);

	var response = await httpClient.SendAsync(request);

	data.StatusCode = response.StatusCode;
	if (response.StatusCode == HttpStatusCode.OK)
	{
		var content = await response.Content.ReadAsStringAsync();
		var parsed = JObject.Parse(content);

		data.ApplicationsUrl = (string)(parsed["_links"]["applications"].First as JProperty).Value;
	}

	await HandleState(data);
}
```

*   Authenticate to UCWA using the OAuth token from the previous step.
*   Retrieve the URL that will be used in the next step.

### CreateApplication()

We must register/create an application on the UCWA server so that we can call API functions or receive event notifications.

```csharp
private async Task CreateApplication(StateData data)
{
	var postData = new
	{
		UserAgent = "IotPresence",
		EndpointId = Guid.NewGuid().ToString(),
		Culture = "en-US"
	};
	var request = new HttpRequestMessage()
	{
		RequestUri = new Uri(data.ApplicationsUrl),
		Method = HttpMethod.Post,
		Content = new StringContent(JsonConvert.SerializeObject(postData), Encoding.UTF8, "application/json")
	};

	if (CheckIfSameDomain(data.ApplicationsUrl, data.OAuthUrl))
		request.Headers.Authorization = new AuthenticationHeaderValue(data.ApplicationTokenType, data.ApplicationToken);

	var response = await httpClient.SendAsync(request);

	data.StatusCode = response.StatusCode;
	if (response.StatusCode == HttpStatusCode.Unauthorized)
	{
		// We are on a split-domain scenario. We need to re-authenticate with the new oauth url
		currentState = 1;
		data.WwwAuthenticate = response.Headers.WwwAuthenticate.ToString();
	}
	else if (response.StatusCode == HttpStatusCode.Created)
	{
		// New application created
		var content = await response.Content.ReadAsStringAsync();
		var parsed = JObject.Parse(content);

		// hoard this for later
		this.cache.BaseAddress = response.RequestMessage.RequestUri.GetComponents(UriComponents.SchemeAndServer, UriFormat.Unescaped);
		this.cache.Application = JsonConvert.DeserializeObject((parsed["_links"]).ToString());
		this.cache.Me = JsonConvert.DeserializeObject((parsed["_embedded"]["me"]).ToString());
		this.cache.People = JsonConvert.DeserializeObject((parsed["_embedded"]["people"]).ToString());
		this.cache.OnlineMeetings = JsonConvert.DeserializeObject((parsed["_embedded"]["onlineMeetings"]).ToString());
		this.cache.Communication = JsonConvert.DeserializeObject((parsed["_embedded"]["communication"]).ToString());

		// Next step
		data.BaseAddress = this.cache.BaseAddress;
		data.MakeMeAvailableUri = this.cache.Me.Links.MakeMeAvailable.Href;
	}

	await HandleState(data);
}
```

*   Prepare our application details to be posted to the application URL.
*   Post the application details serialized as JSON.
*   If our application URL and OAuth URL are the same domain, then we can send our OAuth token as the authentication header.
*   If the Application URL and OAuth URL domain do not match, then we expect to get challenged to authenticate again and reset the state machine back.
*   When the application is created, we deserialise the data returned and cache it for our application to use.

### MakeMeAvailable()

We need this step so that we can receive events. When I left this step out, there was much head scratching.

```csharp
private async Task MakeMeAvailable(StateData data)
{
	if (!this.authenticated)
	{
		string[] modalities = new string[] { };
		var postData = new
		{
			SupportedModalities = modalities
		};

		var request = new HttpRequestMessage()
		{
			RequestUri = new Uri(String.Format("{0}{1}", data.BaseAddress, data.MakeMeAvailableUri)),
			Method = HttpMethod.Post,
			Content = new StringContent(JsonConvert.SerializeObject(postData), Encoding.UTF8, "application/json")
		};

		if (CheckIfSameDomain(data.ApplicationsUrl, data.OAuthUrl))
			request.Headers.Authorization = new AuthenticationHeaderValue(data.ApplicationTokenType, data.ApplicationToken);

		var response = await httpClient.SendAsync(request);
		data.StatusCode = response.StatusCode;

		await HandleState(data);
	}
	else
	{
		data.StatusCode = HttpStatusCode.NoContent;
		await HandleState(data);
	}
}
```

*   Prepare list of supported modalities. In my case it is blank as I'm only interested in the users presence.
*   Post serialised data to "MakeMeAvailable" URI.
*   Recursive call to HandleState.

## Demo

A quick video of the authentication service used to get Skype for Business (Lync) presence from the server (with Events) using a Raspberry Pi 2 with Windows 10 IoT.

{% include embed/youtube.html id='9MhUzSouJgs' %}

## Download

[UCWA Authentication Service](/assets/misc/authenticationservice.zip) (C#)