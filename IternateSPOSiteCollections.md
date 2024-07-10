There are scenarios that we need to iternate SPO site collections for specific purpose. And we need a super admin account to iterate all SPO site collections.
To bypass the limitation, the sample script will use App only permission to iterate all SPO site collections rather than granting a super admin accout to all SPO site collections.
A revised version for [Find All sites where "Everyone except external users" is Added](https://www.sharepointdiary.com/2024/02/remove-everyone-except-external-users-from-sharepoint-online-site.html)


Step 1:
Register an Azure application follow the demo [here](https://github.com/pnp/PnP-PowerShell/tree/master/Samples/SharePoint.ConnectUsingAppPermissions)

```
#STEP2 Export the site collection needs to be checked->SharePoint admin user credential
#Admin centerURL
$filepath="C:\temp\AllSites.csv"
$AdminPortalUrl="https://m365x17384749-admin.sharepoint.com/"

##########Check for all sites############
Connect-PnPOnline $AdminPortalUrl -Interactive
# Get all SharePoint Online sites
$AllSites = Get-PnPTenantSite | Where -Property Template -NotIn ("SRCHCEN#0", "REDIRECTSITE#0", "SPSMSITEHOST#0", "APPCATALOG#0", "POINTPUBLISHINGHUB#0", "EDISC#0", "STS#-1")

# Export the list of sites to a CSV file
$AllSites | Select Url,Template,LocaleId| Export-Csv -Path $filepath -NoTypeInformation


#STEP3 Iterate the site collection needs to be checked->App Only credential
$Allsites = Import-Csv $filepath 

$ResultSiteList = @()
$ResultFile = "C:\temp\EveryoneExceptGrp.csv"

Foreach($site in $AllSites)
{

	#Start-Sleep -Seconds 5
    
    Write-host "Processing Site:"$site.URL -ForegroundColor green    
    
    #Connect to PnP Online
    Connect-PnPOnline -Url $Site.URL -ClientId 526bfdd0-d385-4577-ac5e-96dc1d36ce6b -Tenant 'm365x17384749.onmicrosoft.com' -Thumbprint bd1007903420ce4c277aa5f62f77daee2d317b9c

    $Group = Get-PnPUser -WithRightsAssigned |where {$_.title -eq "Everyone except external users"}

    If($Group)
    {
        $ResultSiteList += $site.URL
        Write-host "`t Everyone except external users group is in this site: $($site.URL)" -ForegroundColor Yellow
    }    
}

##########Export the result to csv#########
#Write-host $ResultSiteList
$ResultSiteList |out-file $ResultFile
