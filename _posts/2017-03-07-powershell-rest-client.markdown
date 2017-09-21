---
title:  "Using Powershell as a REST Client"
date:   2017-03-07 08:20:11 +1200
published: false
---
In the brave new world ushered in by Microsoft as part of their aggressive push toward Azure services, Powershell is quickly becoming a relevant and necessary tool in the systems administrators kit. Now that even Microsoft are exposing their products configuration via Powershell APIs and declaring that going forward their tools will be "Powershell First". Powershell is becoming as ubiquitous in the Windows world as SSH & Bash are in the Unix sphere.

## Querying remote API Endpoints
A common task for managing a remote system is to send some kind of call to an API. Many services will only expose a subset of their configuration capability in their management portal, leaving the more esoteric capabilities to a web-based API. Tools like Postman or curl are useful for one-off calls, or for testing purposes but can be cumbersome for day-to-day use.

## Creating a Cmdlet for API Calls
In order to automate these tasks, we will create a Powershell cmdlet for the purposes of sending a request to the API. Before we do so, it is useful to get an idea of the basic workflow of our task, which is as follows:
1. Send an authentication request to the login endpoint and obtain an API token
1. Use this API token to perform some actions
1. Once complete, send the token to the logout endpoint

In the `begin` section of our cmdlet, we will perform the authentication tasks as required. The `process section` will allow us to pipe in tasks to be performed and finally, the `end` section will use the token to log out.

<figure>
```python
def main(args):
    print("This is a code block!")
```
	<figcaption>
		Jekyll allows plain HTML to be generated from a group of markdown files
	</figcaption>
</figure>

```powershell
function Get-nlUserPermissions
{
    [CmdletBinding(SupportsShouldProcess=$true,
                   PositionalBinding=$false,
                   DefaultParameterSetName="credential",
                   ConfirmImpact='Low')]
    [OutputType([System.Management.Automation.PSCustomObject])]
    Param
    (
        # The usename to fetch the details from
        [Parameter(Mandatory=$true,
                   ValueFromPipelineByPropertyName=$true,
                   Position=0)]
        [ValidateNotNullOrEmpty()]
        [string]
        $Name,

        # An API token for making authenticated calls
        [Parameter(Mandatory=$true,
                   ValueFromPipeline=$false,
                   ValueFromPipelineByPropertyName=$false,
                   Position=1,
                   ParameterSetName="token"
        )]
        [ValidateNotNullOrEmpty()]
        [string]
        $Token,

        # A Username/Password pair to pass to the API to authenticate
        [Parameter(Mandatory=$true,
                   ValueFromPipeline=$false,
                   ValueFromPipelineByPropertyName=$false,
                   Position=1,
                   ParameterSetName="credential"
        )]
        [ValidateNotNullOrEmpty()]
        [System.Management.Automation.PSCredential]
        $Credential
    )

    Begin
    {
        $AUTH_ENDPOINT = 'http://example.com/api.php?Task=Login&Username={0}&Password={1}'
        if($PSCmdlet.ParameterSetName -eq 'credential')
        {
            $url = ($AUTH_ENDPOINT -f $Credential.GetNetworkCredential().UserName,
                    $Credential.GetNetworkCredential().Password)
            $result = Invoke-RestMethod -Method Get -Uri $url
            ### Check the result status and extract the API token
            $Token = $result.Response.Token
        }
    }
    Process
    {
        $TARGET_ENDPOINT = 'http://example.com/api.php?Token={0}&Task=DoWork&Name={1}'
        $url = ($TARGET_ENDPOINT -f $Token, $Name)
        $result = Invoke-RestMethod -Method Get -Uri $url
        ### Check the result status and extract the information needed
        [xml] $userObject = $result.Response.User
        Write-Output $userObject
    }
    End
    {
        $LOGOUT_ENDPOINT = 'http://example.com/api.php?Task=Logout&Token={0}'
        $url = ($LOGOUT_ENDPOINT -f $Token)
        $result = Invoke-RestMethod -Method Get -Uri $url
        ### Ensure that the result status is OK, report an error if not
    }
}
```
