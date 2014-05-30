#Yammer Anywhere In Salesforce.com
##Why? 

Want to integrate Yammer and Salesforce.com? Yammer has developed a JavaScript SDK which allows you to easily integrate any Yammer feed to Salesforce with very few lines of code. This is ideal for any company looking to make their internal software more tightly integrated.

##Yammer And Chatter 
Yammer is built around the idea of users, groups, and feeds. A user belongs to many groups, which have their own individual feeds. Everyone belongs to the 'All Company' group by default. Our example Yammer domain has Sales, Marketing, Accounting, and Engineering groups.

Salesforce (with Chatter) is composed of users, objects (fancy tables), records, and feeds. Salesforce specializes in CRM data, and with Chatter, everything is related to a set of data or specific record. There is another layer of granularity with Chatter because it is forced to reference CRM data. Chatter dictates the need for Salesforce, which isn't always the best tool for everyone in your organization.

Yammer can be the social layer regardless of your CRM solution.

##The Story
+ Bill Nye, Science Guy, has customers and partners. From Salesforce, Bill wants to check with his accountants (who use only Yammer) to ensure that Neil Tyson received the invoice. He messages Ken Gene in the accounting department, "Hey Ken, when did we send out the invoice to Neil?"
+ Ken answers from Yammer, "Let me check"
	+ "He downloaded the invoice in March"

####In Salesforce:

![yammer conversation](img/yammer_story_account_sfdc.png?raw=true)

##Guide For Setup

###Yammer Sign Up
The first thing you need to start is a [Yammer domain](https://www.yammer.com/?return_home=true). We'll assume you already have this setup since you are still reading, so let's get started.
+ Login to your Yammer domain
+ Create a group - Use 'Accounting' as an example (reference this group later)

![sht1](img/ScreenShot1.png?raw=true)

####Create A Yammer App
+ [Guide](http://developer.yammer.com/introduction/#gs-registerapp) to getting started
+ Navigate ['by clicking this link'](https://www.yammer.com/client_applications)
+ 'Register New App'
	+ App Name: salesforceyammin
	+ Organization: Example LLC
	+ Support email: help@examplellc.com
	+ Website: http://www.examplellc.com
+ Done!

![sht2](img/ScreenShot2.png?raw=true)

You need the Client ID when creating the HTML/Scripts of the app in Salesforce later, so just keep it handy. In order to use the Yammer Embed API, you'll have to specify your 'JavaScript Origins' and 'Redirect URI' in the 'Basic Info' section of the app. You don't know the URLs yet, so we'll be coming back to that later.

JavaScript Origins is for CORS. [CORS](http://www.html5rocks.com/en/tutorials/cors/) is an important part of the JavaScript SDK. In summary, when using client side JavaScript to make callouts to external sources (XHR to Yammer API), the requested server has to be able to accept Cross-Origin requests. We won't go into more detail, but it is important if you plan to use the REST capabilities of the SDK.

Now you're ready to create a place for your Yammer application to live in Salesforce.com.

###Salesforce Sign Up
Create a new [salesforce instance (DE)](https://developer.salesforce.com/)
+ Option 1: Sign Up -> fill out form -> Sign me up
+ Option 2: Use your existing Salesforce.com sandbox


####Salesforce Setup

[Quick start VF page doc](http://www.salesforce.com/us/developer/docs/pages/Content/pages_quick_start_hello_world.htm)
<br/>
[Full VF doc](http://www.salesforce.com/us/developer/docs/pages/index.htm)

#####Create Visualforce Page
Once logged into your new Salesforce.com org:
+ Option 1: From the Salesforce UI
	+ Setup -> Develop -> Pages
	+ Click the `New` Button
	+ Name the page 'accountyammer'
+ Option 2: Our preferred Salesforce IDE is [Mavens Mate for Sublime Text](http://mavensmate.com/)
	+ cmd+shift+p -> 'New Visualforce Page'
	+ Name the page 'accountyammer'

![sht3](img/ScreenShot3.png?raw=true)

Copy and paste this Visualforce code:

		<apex:page showHeader="true" sidebar="true" standardController="Account" >
			Hello Visualforce
		</apex:page>

Preview the page in Salesforce.com. https://SALESFORCE_DOMAIN/apex/accountyammer

####Yammer Widget In Visualforce Page
[Embed API ref](https://developer.yammer.com/connect/)

You now have a Visualforce page set up, so it's time to drop in your Yammer Embed widget. In the Visualforce page, include the Yammer JavaScript SDK. The `data-app-id` is the public token generated when you create your Yammer app. Find the ID through Yammer.com; Created Apps -> app_name -> Client ID.

		<script type="text/javascript" data-app-id="YOUR_CLIENT_ID" src="https://assets.yammer.com/assets/platform_js_sdk.js"></script>

The first thing to do after you have the widget is to authenticate with the Yammer servers. [Yammer Client-side OAuth 2](http://developer.yammer.com/authentication/#a-button)
		
		<script> 
			yam.connect.loginButton('#yammer-login', function (resp) { 
				if (resp.authResponse) { 
					console.log('pass');
				}
				else {
					console.log('fail');
				}
			}); 
		</script>
		<span id="yammer-login"></span>

Refresh the page in your browser. You should receive a JavaScript exception in your console (When testing with Chrome and Mac, cmd+alt+j toggles the JavaScript Console). To fix, navigate to Yammer:
+ Created Apps -> My Apps -> sfdcyammin 
	+ Basic Info -> Redirect URI
		+ https://c.na15.visual.force.com/apex/accountyammer
	+ Basic Info -> JavaScript Origins
		+ https://c.na15.visual.force.com/


Add the HTML for the embed widget and the JavaScript to render your feed. Get your Accounting `feedId` from the Yammer group URL. Ex: `https://www.yammer.com/examplellc.com/#/threads/inGroup?type=in_group&feedId='0000000'`
		
		<script> 
			yam.connect.loginButton('#yammer-login', function (resp) { 
				if (resp.authResponse) { 
					console.log('pass');
					yam.connect.embedFeed({
						container: '#embedded-feed',
						network: 'examplellc.com', // This must match your Yammer domain
						feedType: 'group',
						feedId: '0000000', //found in the yammer url
						config: {
							use_sso: false,
							header: false,
							footer: false
						}
					});
				}
				else {
					console.log('fail');
				}
			}); 
		</script>
		<span id="yammer-login"></span>
		<div id="embedded-feed" style="height:400px;width:500px;"></div>
		...

Your Yammer feed should now load automatically after you've logged in once through Yammer. We are setting the `use_sso` attribute because we are using the `loginButton` method to authenticate the user. If your organization is Yammer Enterprise with SSO (Single Sign-On) on your network, you can utilize this property to delegate authentication.

####Configure Account Detail Override
The Embed component works best with some limited context. It would be nice to see the feed while looking at an Account record in Salesforce.com. You can accomplish this with Visualforce pre-built components, specifically the ['detail' component](ref: https://www.salesforce.com/us/developer/docs/pages/Content/pages_compref_detail.htm). 
		
		<apex:detail subject="{!account.Id}" />

As of now you have a standalone page showing your Yammer feed. You need to override the account object's 'view' to show the Yammer feed within the context of your account record. In Salesforce.com:

+ Navigate to Setup -> Customize -> Accounts -> Buttons, Links, and Actions
+ Edit the action labeled 'View' 
+ Override with your Visualforce page, 'accountyammer'. Note: it is available in the drop down because you defined your Visualforce page to have the 'Account' as the standardController attribute.
+ If you don't have a record in the Account object, you can create one now. Navigate to the Accounts tab and click New.
+ Otherwise, navigate to an Account record and you should see your Yammer feed 'embedded' in your Account detail page.

![sht4](img/ScreenShot4.png?raw=true)

This works fine, but it would be nice if you could toggle the feed when we want to only see the Account details. To do this, wrap the 'embedded-feed' tag in a 'div' and add an element we can click.

		<div id="feed">Yammer Feed</div>
		<div id="feedHolder">
			<div id="embedded-feed" style="width: 100%; height: 300px;"></div>
		</div>

You can make this super easy by including jQuery to toggle the display of the feed. jQuery is a bit overkill, but is fine for this demo.

		<script type="text/javascript" src="https://code.jquery.com/jquery-2.1.1.min.js"></script>
		<script> 
			//yammer sdk uses jquery as a dependency, so I want to eliminate any possible conflicts
			var $j = jQuery.noConflict();
			//necessary for adding event handlers after dom elements loaded
			$j( function() {
				$j("#feed").click( function() { $j("#feedHolder").toggle() });
			});
			...
		</script>

Your 'accountyammer' Visualforce page should look like this in the end:
		
		<apex:page showHeader="true" sidebar="true" standardController="Account" >
			<script type="text/javascript" data-app-id="YOUR_CLIENT_ID" src="https://assets.yammer.com/assets/platform_js_sdk.js"></script>
			<script type="text/javascript" src="https://code.jquery.com/jquery-2.1.1.min.js"></script>
			<script> 
				//yammer sdk uses jquery as a dependency, so I want to eliminate any possible conflicts
				var $j = jQuery.noConflict();
				//necessary for adding event handlers after dom elements loaded
				$j( function() {
					$j("#feed").click( function() { $j("#feedHolder").toggle() });
				});
				yam.connect.loginButton('#yammer-login', function (resp) { 
					if (resp.authResponse) { 
						console.log('pass');
						yam.connect.embedFeed({
							container: '#embedded-feed',
							network: 'examplellc.com',
							feedType: 'group',
							feedId: '0000000',
							config: {
								use_sso: false,
								header: false,
								footer: false
							}
						});
					}
					else {
						console.log('fail');
					}
				}); 
			</script>
			<span id="yammer-login"></span>
			<div id="feed">Yammer Feed</div>
			<div id="feedHolder">
				<div id="embedded-feed" style="width: 100%; height: 300px;"></div>
			</div>
			<apex:detail subject="{!account.Id}" />
		</apex:page>



###Open Graph
Assume you only want to show a conversation relevant to the context of a specific record in Salesforce. Open Graph allows you to capture the context of a URL in Salesforce.com. If you have an Open Graph feed on an Opportunity page, the feed will only show messages created in context of that URL. This is a great feature of the Yammer API.

####Opportunity Context, Visualforce Page
Follow the same steps for 'Create Visualforce page' in the previous portion of setup.
+ Name the page 'opportunityyammer'
+ Copy/paste the visualforce code from 'accountyammer' page
+ Change:
	+ `standardController="Opportunity"`
	+ `container: "#og-feed",`
	+ `feedType: "open-graph",`
	+ Add to config JSON `promptText: "Comment on this deal",`
	+ `<div id="feed">Open Graph Feed</div>`
	+ `<div id="og-feed" style="width: 100%; height: 300px;"></div>`
	+ `<apex:detail subject="{!Opportunity.Id}" />`

Now it's time to override the Opportunity view. Follow the same steps as previously defined for the 'Configure Account Detail Override' section, but for the Opportunity object.
+ Navigate to the Opportunity tab
+ Select an existing Opportunity (create one if you're in a fresh Salesforce instance)

A new type of feed should be present on the Opportunity page. Any message typed in this feed, will only be relevant to this Opportunity. The Open Graph feed accomplishes this by referencing the URL of the page.

![sht1](img/ScreenShot5.png?raw=true)

####Unique URLs
It is important to point out, if the URL changes in any way, you won't see any information.
To demonstrate:
+ Type and post a message in this Opportunity (Open Graph) feed
	+ This message will be posted to Yammer with a link back to this Opportunity
+ Change the URL parameter to '2', `sfdc.override=2`
+ Refresh the page
+ Notice the feed for this Opportunity is blank

Giving context to your conversation is easy. Just change a few lines of code and Yammer will automatically reference your URLs.

The demo app is now complete. Yammer's API has a lot more to it than just the Embed widget, so be sure to check out all the documentation.

##Balancing Context
Context awareness (with Yammer in Salesforce) requires an understanding of the Open Graph technology in the Yammer API. OG is not only for pushing 'activities' into Yammer for display in the 'Recent Activity' widget. This would be useful on its own, but the real power of Open Graph is in the feed, because it's about the conversation and maintaining the context of a conversation.

##You're Done!
We think this is a great start for anyone interested in beginning Yammer development on the Salesforce platform. Let us know if you have any questions or comments.

Happy coding!