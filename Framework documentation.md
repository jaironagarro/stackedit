# Boomi postman test generation framework

## Table of contents

1. [Test abstractions](#test-abstractions)
    1. Base Script
    2. Base Test
    3. Variables
    4. Transformation
    5. Test
2. Usage
3. Troubleshooting
 
## Test abstractions
### Base Script
The base script is the raw base for the pre-request script to be assemble for the postman request. It must always have the next code in the beginning of the file

    var body_str = JSON.stringify(body);  
    pm.environment.set('request_body', body_str);
This script could be used to set environment variables to be used in the [Base Test script](#base-test) and could also set up variables and methods that could be used on the Transformations.

This file name is **baseScript.js** and is found in the resources folder at root level.
### Base Test
The base test is the other side of the test assemble as it is the base from which all the tests are assemble and should always make sure that the next code is present at the beginning of the file

    const jsonData = pm.response.json();
This script should be used to put tests that are always in every test and should work with environment variables that are set in the [Base Script](#base-script).

This file name is **baseTest.js** and is found in the resources folder at root level.

### Variables
Variables, as its name suggest, are a way to standardize a [Transformation](#transformation) or a [Test](#test) te be used in many scenarios. Both [Transformations](#transformation) and [Tests](#test) have an space in its format to declare its variables and each one should be in the next format:

    variableName = "Variable description to be  shown whenever the user is setting it"
Each variable should be used in both [Transformations](#transformation) and [Tests](#test) scripts with a leading "$" like this example:

    pm.environment.set('someVariable', $variableName);


### Transformation
The transformations are pieces of code that work with the body in some way to transform the request body, in this case the CLR that is sent but not necessarily for other kinds of tests. This transformations could be used to swap information, to generate values, to find specific values that need to be  put into an environment variable to use in the tests, etc.

Each transformation needs to have a format and to be in its own file named to be recognize easily for its purpose, the transformations format is this:

    "Description of the transformation"  
	/**/  
	Space for variables, in case that no variable is declared in this file it should be null instead
	/**/
	Space for the script

Example with no variables:

    "The leadId is automatically edited to be different from the CLR leadId"  
	/**/  
	null  
	/**/  
	var leadId = body.leadInformation.leadId;  
	for (var i = 0; i < 4; i++) {  
		var position = Math.floor(Math.random() * leadId.length);  
		var replaceValue = Math.floor(Math.random() * 10);  
		leadId = leadId.substring(0, position) + replaceValue + leadId.substring(position + 1);  
	}  
	body.leadInformation.leadId = leadId;  
	pm.environment.set('qa_lead_id', leadId);

Example with variables:

    "Flag and account info in case is necesary to swap this in the test"  
	/**/  
	accountInfoSwap = "If it is necessary to swap the account info in the CLR, this is a boolean value"  
	/**/  
	var accountInfoSwap = false;  
	// Account information for tests, is replaced in the body for testing purposes if flag is active  
	var accountInfo = {}  
	if (accountInfoSwap) {  
		body.accountInfo = accountInfo;  
	}

The transformations to be used for an specific test generation should be in the "./transformations" folder under resources. Any other kind of transformation could be saved in another folder to be used when needed but only those in the transformations folder will be considered.

### Test
Tests work on two stages, the first script space is used to set up the different environment variables to be used in the second script space to be used as a part of the test. Tests should always set up any variable that they will use because there is no way to know how the user could mix and match the tests that are going to be part of a postman test after all.

Each test needs to have a format and to be in its own file named to be recognize easily for its purpose, the transformations format is this:

    "Description of the test"  
	/**/  
	Space for variables, in case that no variable is declared in this file it should be null instead
	/**/
	Space for the first script, the one to set everything up
	/**/
	Space for the second script, the one where te postman tests are being executed.

Example with variables:

    "Validate customMapping value is correct"  
	/**/  
	siebelCode = "Siebel code to be expected on the custom mapping"  
	/**/  
	var siebelCode = '$siebelCode';  
	pm.environment.set('qa_siebel_code', siebelCode);  
	/**/  
	pm.test("Validate customMapping value is correct", () => {  
		pm.expect(jsonData.customMapping).to.eql(pm.environment.get('qa_siebel_code'));  
	});
The tests to be used for an specific postman test generation should be in the "./tests" folder under resources. Any other kind of test could be saved in another folder to be used when needed but only those in the tests folder will be considered.

## Usage
Under the "./clrs" folder within resources should be one CLR file for each postman test to be created with the name of the postman test to be generated, this could be for instance the name of the source that is being tested.

Next, the user should select transformations and tests that are going into their respective folders accordingly to what is expected to be tested with the postman test.

Finally the user should run the MainTest file to be prompted about any variable that is needed for the program to finish.

Once the program has finished in the "./classes/out" folder under target in the root project folder should contain a folder for each CLR that was input to the postman test generation. Each folder should contain  two files that has the name of the CLR and a trailing "-pre" and "-post". The "pre" file is the one which contents should go into the Pre-request script tab of the postman request and the "test" file contents should be in the Tests
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNTYyNjE4ODIsODMxMzU2NjgyXX0=
-->