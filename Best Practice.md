# ServiceNow Centre of Excellence - Best Practice Guide

## Introduction

This document details the ServiceNow best practice for use with the Centre of Excellence.  These are the standards to be applied when working on an implementation for both customer and internal projects.

The following principles should be followed by all consultants;

- Keep things as simple as possible so that the customer and later consultants can understand what has been done
- Stay as close to the baseline (out-of-the-box) system as possible
- Where possible convince the customer that baseline is often better than customisation

## Existing Standards

The ServiceNow technical best practice guides should be reviewed periodically.  These best practices form the basis of our own internal standards with a few exceptions detailed in this document.

- [Scripting](https://developer.servicenow.com/dev.do#!/guides/rome/now-platform/tpb-guide/scripting_technical_best_practices)
- [Client Scripting](https://developer.servicenow.com/dev.do#!/guides/rome/now-platform/tpb-guide/client_scripting_technical_best_practices)
- [Business Rules](https://developer.servicenow.com/dev.do#!/guides/rome/now-platform/tpb-guide/business_rules_technical_best_practices)
- [Debugging](https://developer.servicenow.com/dev.do#!/guides/rome/now-platform/tpb-guide/debugging_best_practices)
- [Logs and Queues](https://developer.servicenow.com/dev.do#!/guides/rome/now-platform/tpb-guide/logs_and_queues_best_practices)
- [Update Sets](https://developer.servicenow.com/dev.do#!/guides/rome/now-platform/tpb-guide/update_set_tbp)
- [XML Data Transfer](https://developer.servicenow.com/dev.do#!/guides/rome/now-platform/tpb-guide/xml_data_transfer_technical_best_practices)

In addition to these there are existing standards for JavaScript coding which can be used to understand how to write efficient JavaScript.  Be aware that some of these standards are written to include the features available in ECMAScript 6 which is not yet available in ServiceNow which currently only supports ECMAScript 5.

- [Coding Conventions for the JavaScript Programming Language (Douglas Crockford)](https://www.crockford.com/code.html)
- [W3C Javascript best pratice](https://www.w3.org/wiki/JavaScript_best_practices)
- [Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html)

Finally, always review any existing best practice adopted by the customer.

## Configuration vs Customisation

In recent years ServiceNow has pushed customers to stick to baseline as much as possible to assist with upgrades and general maintainability.  Along with that they have also improved the upgrade engine within ServiceNow which makes it 'easier' to manage any customisations.

For reference, ServiceNow defines a Customisation as the following [(KB0553407)](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB0553407);

> A customization is any change to code that is part of the baseline install of a ServiceNow instance.

and a Configuration as

> A configuration is tailoring an instance using ServiceNow best practices and API to meet your requirements without making changes to code that is part of the baseline installation of an instance.

ServiceNow's previous approach was a "modify a baseline script and you own it" attitude which was supported by a process of copy, deactivate and change the copy.  This allowed baseline artifacts to continue to be upgraded as the active flag is ignored during an upgrade.  The side-effect however meant that the modified script/artifact could miss out on any new functionality added by ServiceNow as it is not flagged during the upgrade.

The current approach is as follows (taken from the Scripting Fundamentals training guide);

- understand that the customisation can be costly and should be carefully considered.
- research and understand the existing functionality and API's. Can a no-code approach to the same thing?
- determine if the customisation is actually necessary.  Discuss with the customer if the baseline is actually sufficient and explain the impact of modifying baseline (maintainability, upgrades, technical debt).
- If the baseline artifact needs to be modified then make the change directly to the original, but be ready to review and revert the skipped file, if needed, after an upgrade.

When modifying a baseline record make sure to add comments explaining the rationale behind the change and include a reference to the story/incident driving the change, or for non-script based artificacts update the description.

## Custom tables and fields

Creating new custom tables or adding new fields to baseline tables can impact the customers subscription.  Before creating new tables/fields always check with the customer that they have sufficient licenses available.

Review [KB0854339](https://support.servicenow.com/kb?id=kb_article_view&sysparm_article=KB0854339) for an overview of custom table licensing.  Additionally, there is a [custom table guide](https://www.servicenow.com/content/dam/servicenow-assets/public/en-us/doc-type/legal/custom-table-guide.pdf) which deta.

Where possible look to exploit the exempt tables when creating new tables (i.e. use dl_matcher for reference data).

ServiceNow best practice states that custom tables should be created inside scoped applications to aid with subscription management.  This allows the tables to be allocated as a bundle to a suitable subscription.

## Messages

The **sys_ui_message** table can be used to hold any message used within the platform.  Instead of hardcoding messages within your code you should utilise this table along with the supporting API's.  The benefits are as follows;

- messages can be modified in the future without the need for a developer to find the reference in the code, make the change and go through a release process.
- messages can be easily translated into different languages as the table supports multiple entries with a different language code.

Accessing the messages is done using the API's below dependent on where in the platform you are using the message.

### Server Script

    gs.getMessage("message_key");

Server scripts can also use parameters with the message as follows;

    gs.getMessage("Message with a {0}, and {1}", ["parameter", "another one"]);

    // results in the message...
    // Message with a parameter, and another one


### Client Script

    getMessage("message_key");

Client scripts also have a **Messages** field which needs to have each message key listed that is used by the script.  This preloads the messages to improve performance.

You can also use parameters with messages on client scripts using the following code;

    var msg = getMessage("Message with a {0}, and {1}");
    msg = msg.withValues(["parameter", "another one"]);

    // results in the message...
    // Message with a parameter, and another one

### Widgets

Within widget HTML you can use the following format.  This does not support parameters in messages, so if that is required you should generate the message in the client script.

    ${message_key}

And from the client controller, the i18n library can be used using the following code;

    function($scope, i18n) {
        var msg = i18n.getMessage('Message with a {0}, and {1}')
            .withValues(["parameter", "another one"]));
    }

## Update Sets

### Naming and description

Update sets should include the prefix CC along with a reference to the story/task driving the update.  

- CC [task number] [developer initials] [Description]
- CC STRY001245 AC Description of story

The description field should also be populated with details of the change.

### Batch vs Merge

Merging of update sets used to be the preferred way of releasing groups of update sets between instances.  The process involved selecting the update sets and merging all update records into a single update set.  Where an artifact was updated in multiple update sets, only the latest change will be captured.  This single update set could then be exported or retrieved into an instance as a single unit rather than having to manage a group of update sets.  One of the cons of merging was that the context of the change was ultimately lost.

The Jakarta release introduced Batch Update sets.  This allows for update sets to be grouped in a parent/child hierarchy and moved as a single batch between instances.  The benefits of this are;

- the context of a change are not lost as the original update set is retained
- update sets can be grouped into a logical hierarchy such as Release, Change and Fixes
- an update set can be removed from the batch allowing for last minute changes to the release

Further information on batching can be found here: [Update set batching](https://docs.servicenow.com/bundle/rome-application-development/page/build/system-update-sets/hier-update-sets/concept/us-hier-overview.html#us-hier-overview)

## Naming Standards

### Design Artifacts

Design artifacts such as business rules, client scripts, widgets, script includes etc should be prefixed with CC to indicate that the element has been created by the CoE. Additionally the name should reflect the purpose of the artifact.

    CCUpdateIncidentState
    CCValidateUserInput

**UI Policies** do not have a name so the description field should be used instead.

### Script Includes

ServiceNow loosely adopts the following standard for naming script includes;

- if the script include contains utility methods for a table add the Utils suffix; **CCTableNameUtils**
- if the script include is client-callable add the Ajax suffix; **CCLabelValidationAjax**

### Variables

Variables should use camel case and reflect the purpose of the variable.  Additionally, if related to a Glide* class then a suitable prefix applied to the name.  

    grUser      (GlideRecord for sys_user)
    gdtToday    (GlideDateTime for current date/time)
    userName
    incidentCount

Avoid using short and meaningless names with the exception of loop variables where 'i', 'j' etc are acceptable.

Constants are variables whose value cannot be changed once assigned.  Constants in JavaScript were not introduced until ES6 and so are not fully supported in ServiceNow, however the standard for naming a constant is to have it entirely UPPERCASE.

### Tables

Table names and labels should always be singular (i.e. Incident - incident, Incident Task - incident_task)

Table names should be written in snake case where each space in the name is replaced by the underscore character (i.e change_task)

Custom tables will be prefixed with u_

### Fields

Field labels should be in sentence case where only the first word is capitalised (i.e. Configuration item).  This is despite plenty of examples in the baseline system where this convention is not followed.

Field names should be written in snake case similar to table names.

Custom fields will be prefixed with u_

### UI Actions

Within a UI Action the **action name** should be prefixed with cc_ to prevent any clashes with actions in the baseline system.

## Comments

The following advice should **not** be followed.

> True programmers do not comment their code.
> If it was hard to write, then it should be hard to read.

Comments help to explain complicated code and why code has been updated (linking to a story for example) amongst other reasons.

JavaScript supports two styles of comments.

    // single line comments

    /*
     Multiple line or
     Block comments
    */

If you are changing a baseline object then always add code explaining what you have changed or added.  This allows the changes to be reviewed during an upgrade and a decision made on keeping the changes or reverting back to out of the box.

Don't add unncessary comments, such as those that explain obvious code, i.e.

    i = i + 1;  // add 1 to i

Instead, use comments to explain the rationale behind a change, or to explain 'advanced' coding techniques, for example when using GlideQuery it might be useful to explain some of the parts of the call.

    var gqUserHasRole = new global.GlideQuery('sys_user_has_role')
        .where("user", userSysId)           // build the query - user=sys_id^role.name=role_name
        .where("role.name", roleName)
        .count();                           // return the count of records matching the query

Or a more complex example;

    var gqDirectReports = new global.GlideQuery("sys_user")
        .where("active", true)
        // build a new query for detecting manager, allows us to implement an ^OR query
        .where(new global.GlideQuery().
            where("manager", managerSysID)
            .orWhere("u_functional_manager", managerSysID))
        // only need name, email and the display value of location
        .select('first_name', 'last_name', 'email', 'location$DISPLAY')
        // add the full_name property based on first and last name
        .map(function (user) {
            user.full_name = [user.first_name, user.last_name].join(' ');
            return user;
        })
        // the reduce method will create a usable array (alternative to the .toArray() method)
        .reduce(function (arr, user) {
            arr.push(user);
            return arr;
        }, []);

If you comment out existing code then you should also add a comment explaining why.

### Script Include Methods

JSDoc style comments should be used to document methods in script includes.  These are a structured comments which can be used to generate documentation if needed (using third-party tools such as the [SNDoc plugin](https://developer.servicenow.com/connect.do#!/share/contents/6426327_sndoc?v=1.1&t=PRODUCT_DETAILS))

An example JSDoc comment is as follows;

    /**SNDOC
      @name _parseComment
      @private
      @description Parses over an individual comment with the tag parsers
      @param {string} [comment] - a single comment as a string to parse over
      @returns {object} An object will properties for each of the tags in the comment
    */
    _parseComment: function(comment){
        /* some code */
    },

The comment gives details of the purpose of the method, the expected parameters along with the return value.

## Client Scripts

Always check to see if UI Policies can be used instead of client scripts.

Limit the number of client scripts for a table as these have an impact on load times of lists and forms.

Avoid DOM (Document Object Model) manipulation.  It can cause a maintainability issue when browsers are updated. The only exception is when you are in charge of the DOM: in UI Pages, and the Service Portal (taken from ServiceNow Best Practice)

By default client scripts are run in Strict mode and as such do not have direct DOM access.  Access to jQuery, prototype and the window object are disabled.

If you do need to get access to a form element then use the following;

    g_form.getElement('field_name');

### Server interaction from Client Scripts

Avoid synchronous server calls as these block the client and can degarde the user expererience.

GlideAjax is the preferred method for making server calls.

Use **before** business rules and **g_scratchpad** to load server data on page load where possible.

If you must use getReference then always use it with a callback but be mindful that the entire record is returned which is not always needed.

The following table (taken from the ServiceNow developer community) shows the different ways of interacting with the server along with the simplicity/efficiency.

| API | Simplicity | Efficiency | Description |
| --- | ---------- | ---------- | ----------- |
| GlideRecord getReference | 1st | 5th | 1 line of code, poor UX (blocks browser), returns entire record |
| GlideRecord | 2nd | 4th | ~5 lines of code, poor UX (blocks browser), returns entire record |
| GlideRecord getReference (with callback) | 3rd | 3rd | 5-10 lines of code (with callback), best UX (does not block browser), returns entire record |
| GlideRecord (with callback) | 4th | 2nd | 10 lines of code (with callback), best UX (does not block browser), returns entire record |
| GlideAjax | 5th | 1st | ~20 lines of code (client script + script include), best UX (does not block browser), returns only the data you need |


## JavaScript Coding Standards

Coding styles with any programming language can be subjective and JavaScript is no exception.  In general the standards detailed in the links at the start of the document along with those here should be followed in all code delivered by the CoE.  High level guidelines are;

- try and write self documenting code by using comments and meaningful variable names.
- write clean readable code and be consistent with your coding style
- advanced coding techniques or new features do not have to be avoided (such as ternary operators and GlideQuery for example), just ensure that the code is readable and commented.

Code should always be formatted correctly to aid readability.  

The following code highlights some of the good practice discussed.

    (function () {
    	var ouList = '';
        var ouListArr = [];
    	// Get a list of the sources and iterate through them
    	var sourceList = current.source.toString().split(',');
    	for (var i=0; i< sourceList.length; i++) {

		    // if the source is an OU, then add it to the ouList
		    if (sourceList[i].indexOf('OU=') == 0) {
                ouList.push(sourceList[i]);
		    } else {
    			// Do something else
		    }

	    }
	    
        ouList = ouListArr.join(','); // create comma seperated list of OU's
    }());

The following best practice are shown in the code;

- indentation within **function**, **for** and **if** blocks.
- spacing following keyswords
- use curly braces on if statements (even if single line of code)
- else on the same line of the closing curly brace of the initial if block

An alternative for the formatting of the **else** statement is to start this on a new line.  This can help with code collapsing tools available in code editors.  For example;

    if (sourceList[i].indexOf('OU=') == 0) {
        ouList.push(sourceList[i]);
	}
    else {
    	// Do something else
	}

### Formatting code

The ServiceNow syntax editor includes a **format code** button which will attempt to format your code based on the JavaScript standards.

It is possible to integrate with other code editors such as Visual Studio Code using [SNUtils](https://www.arnoudkooi.com/) or the [ServiceNow VS Code extensions](https://docs.servicenow.com/bundle/rome-application-development/page/build/applications/concept/vs-code.html).


## Testing

Automated Test Framework should be at the heart of our development practices.  This helps technical consultants to assess acceptance criteria and use these as the baseline for what their development work will achieve.
 
The result is to continue to aim for high quality implementations and to leave our customers with a simple path for future testing and a clear record of testing we have performed.

Whilst it takes longer to create an ATF than to run a test manually once, after manually testing a few times then the time spent quickly mounts up. As Technical Consultants become more proficient with ATF, then the time to write tests will decrease. This should lead to an overall saving of time rather than a cost, especially as running a test 4 times during development would be common.

As ATF tests can also be used for UAT, then developer-created tests can become dual purpose and contribute to saved effort in UAT.

Running ATF tests provides clear documentation of what tests were run when and provides a statement of how functionality should be tested (which is useful additional documentation of how functionality was intended to be used).

Finally the focus of development must be to implement the stories that have been specified and thus the acceptance criteria resulting from stories. Using the principles of Test-Driven Development reinforces these desired outcomes and ensures that design of development work is thought about in advance.

### Guidelines

- Automated Test Framework (ATF) should be used to assist unit testing.
- A [Test Driven Development (TDD)](https://en.wikipedia.org/wiki/Test-driven_development) approach should be adopted, this requires the tests to be defined before development starts []
- Where changes are required to the data model then ideally this should be identified and implemented before the tests are created (otherwise tests cannot easily be created that use these new tables or fields).
- Tests should be written using the acceptance criteria as a starting point. Acceptance criteria will typically be geared towards the UAT phase and could be too high level to define proper unit tests.  
- Write small tests and group into a test suite if necessary.
- Tests should avoid scripting where possible - i.e. use standard test configuration elements instead.
- Testing should consider any impact on existing functionality (i.e. in the case of a script include, is anything else using the library that could be affected by the change). Check to see if any tests already exist and include these in the test suite.
- When taking ownership of a baseline element, a test should be created so that future changes to the element (i.e. it is reverted back to baseline) ???
- When writing the tests, provide both a pass and fail scenario.  The pass scenario should be run before any development takes place (expecting a fail outcome).
- In the case of large development tasks try to write small unit tests that prove the work as it progresses, this can then form a more complete test at the end.
- When creating tests, the state and result of previous tests should not be relied upon. ATF tidies up any changes made during a test run so cannot be used in subsequent tests.
- ATF can be used to test both server and UI side changes. However, the UI testing capabilities are limited in earlier versions and so can have issues with heavily customised instances. Madrid has improved UI testing.  
- To aid with unit testing, provide a method in Ajax script includes that can be called directly removing the need to simulate the Ajax call or to have to test via the UI.
- Ensure UI testing is performed using the standard operating environment of the customer (e.g. Internet Explorer 11, Windows 10). ServiceNow has known performance issues with older browsers.
- Always impersonate a user with only the privileges needed.  This should be the first step of any ATF script. Create the user if required as the existence of a suitable user cannot be guaranteed.

