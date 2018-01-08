---
layout: post
section-type: post
category: tech
tags: [ 'robot framework,chrome headless' ]
title: "Using Robot Framework with Chrome headless"
description: Using Robot Framework with Chrome headless 
---

**Robot Framework with Chrome headless**

*pre-steps*
- Install RIDE https://pypi.python.org/pypi/robotframework-ride to generate your scripts
** You can use Chrome robot or Fire robot with some modifcations to the resulting script

*Sample Script*
```
*** Settings ***
Library           BuiltIn
Library           Selenium2Library

# close the browser at the end of the script
Suite Teardown    Close Browser

*** Variables ***

*** Test Cases ***
Test Case
    # set options for the web driver
    ${options}=  Evaluate  sys.modules['selenium.webdriver'].ChromeOptions()  sys, selenium.webdriver
    # Add arguments to the browser
    Call Method    ${options}    add_argument      always-authorize-plugins
    Call Method    ${options}    add_argument      enable-npapi
    Call Method    ${options}    add_argument      headless

	  # Create the Web Driver/Browser  passing the options
    Create WebDriver  Chrome    chrome_options=${options}
    
    # Opent the url
    Go to https://www.google.com

    # use wait instead of a pause
    Wait Until Page Contains  Feeling Lucky timeout=12s

    # click the search box
    Click Element    XPATH=//input[@id='lst-ib']

    # send the search text
    Input Text  XPATH=//input[@id='lst-ib'] norco optic 9.1
    # clicking the google search button with type ahead becomes very odd getting the element
    # send a carriage return instead
    Press Key //input[@id='lst-ib'] \\13

    # wait for the result
    Wait Until Page Contains  Optic - Norco Bikes timeout=12s

    # click on an element
    Click Element    //a[contains(text(),'Optic - Norco Bikes')]

```



