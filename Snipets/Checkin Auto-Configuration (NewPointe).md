Checkin Auto-Configuration
==========================

About
-----
This code configures a Check-in station to automatically select all available Check-in Areas (which means yay, no more checking boxes). It will also (on iOS kiosks) save the chosen configuration to the device so it always starts with the proper config.

How it Works
------------
When you go to this auto-config page, it will show you a choice of check-in devices amd themes. Clicking a theme will set the  current theme. Clicking a device set check-in to use that device, automatically select all Check-in areas, and redirect to the check-in screen. If done on an iOS kiosk, it will also save a special auto-config to the device so it always starts with the specified config.

Important Notes
---------------
 - This is built for NewPointe's check-in setup, which means there's only one check-in "Device" per campus. The query here is designed to automatically chose the first device that matches a given campus. 
 - This uses a hard-coded "Checkin Configuration". This means, for example, you can't use the same auto-config page for both Family checkin and Individual checkin.
 - The themes available are hard-coded. You'll want to change those to match your own system.

How to Set Up
-------------
Note: When modifying check-in pages, you might have to add `?Theme=Rock` to the URL to get access to the admin toolbar.

1. Create a new Page on the "Rock Check-in" Site using the "Check-in" Layout.
2. Open up the Page Properties, go to Avanced Settings, and add the following routes (You can use something other than `auto`, just make sure to update it in the lava settings):
   - auto
   - auto/{Campus}
   - auto/{Campus}/{Theme}
3. Add a Dynamic Data Block to the page and configure it as follows:

#### Query

```sql

/*
 * table1 (Auto-config device)
 * Finds the first Device Id related to the passed in Campus Short Code
 */

SELECT TOP 1 d.Id
FROM [Device] d
INNER JOIN DeviceLocation dl ON d.Id = dl.DeviceId
INNER JOIN [Campus] c ON c.LocationId = dl.LocationId
WHERE DeviceTypeValueId = 41 AND LOWER(c.ShortCode) = LOWER(@Campus)

/*
 * table2 (Auto-config check-in areas)
 * Finds all Group Types configured for checkin, and their child groups 
 */

SELECT cgt.Id
FROM [GroupType] gt CROSS APPLY [dbo].[ufnNPCustom_GetChildGroupTypes](gt.Id) cgt
WHERE gt.GroupTypePurposeValueId = 142

/*
 * table3 (Was a valid campus code passed in the URL?)
 * Counts the number of campuses that match the passed in Campus Code
 */

SELECT COUNT(c.Id) AS MatchCount
FROM [Campus] c
WHERE LOWER(c.ShortCode) = LOWER(@Campus)

```

#### Formatted Output

```liquid

{% capture null %}

    AUTO-CONFIG SETTINGS:

    This is the url of the default check-in configuration page. For normal Rock installations, this will be "/checkin".
    {% assign CheckinURL = '/checkin' %}

    This is the page route for this page, it's used to build an auto-config url to save to a device.
    {% assign AutoConfigURL = '/auto' %}
    
    This is a list of themes to show on the auto-config page.
    {% assign ThemeList = 'CheckinNewPointeOrange,CheckinPointePark,CheckinAtTheMovies,CheckinNewPointe' %}
    
    This is the id of the Check-in Configuration to use for all stations.
    {% assign CheckinConfigId = '14' %}

{% endcapture %}

<script>

    // iOS Check-in App Helper Functions (Asynchronous)
    function getApplicationSetting(key, onSuccess, onFail) {
        Cordova.exec(onSuccess, onFail, "ApplicationPreferences", "getSetting", [{ key }]);
    }
    function setApplicationSetting(key, value, onSuccess, onFail) {
        Cordova.exec(onSuccess, onFail, "ApplicationPreferences", "setSetting", [{ key, value }]);
    }
    
    // Saves a given link as the check-in address in the iOS Check-in app
    function saveLink(link) {
    
        // If we're using the Check-in app
        if(Cordova) {
            
            // Try to save the given link (Asynchronous)
            ApplicationPreferencesPlugin.setApplicationSetting('checkin_address', link.href,
                () => window.location.href = link.href,
                (e) => alert(e.message)
            );
            
            // Don't redirect yet (We want to wait until the link is saved)
            return false;
        }
        else {
            // We can't save the link, so go ahead and redirect
            return true;
        }
    }
    
</script>
<style>

.autoconfigbutton {
    color: #333;
    background-color: #fff;
    border-radius: .25em;
    display: block;
    padding: .5em;
    margin: .5em auto;
    font-size: 1.5em;
}

.autoconfigbutton:active,
.autoconfigbutton:hover,
.autoconfigbutton:visited {
    color: #333
    background-color: #eee;
}

</style>


{% assign urlParts = 'Global' | Page:'Url' | Split:'/' %}
{% capture baseURL %}{{ urlParts[0] }}//{{ urlParts[1] }}{% endcapture %}

{% assign kioskId = table1.rows.first %}
{% assign checkinAreas = table2.rows %}
{% assign campusMatchCount = table3.rows.first.MatchCount %}

{% if kioskId != null %}
    {% capture configUrl %}{{ baseURL }}{{ CheckinURL }}?KioskId={{ kioskId }}&CheckinConfigId={{ CheckinConfigId }}&GroupTypeIds={% for area in checkinAreas %}{{ area.Id }},{% endfor %}&Theme={{ PageParameter.Theme }}{% endcapture %}
    {{ configUrl | PageRedirect }}
{% else %}

    {% if campusMatchCount == 0 and PageParameter.Campus %}
        <div class="alert alert-warning"> Could not find a Campus for the given Short Code. </div>
    {% endif %}
    {% if campusMatchCount >= 1 %}
        <div class="alert alert-warning"> Could not find a Kiosk for the given Campus. </div>
    {% endif %}
    
    <div class="text-center">
        <h2>Choose a Campus</h2>
        {% for Campus in Campuses %}
            {% if Campus.IsActive %}
                <a class="autoconfigbutton" href="{{ baseURL }}{{ AutoConfigURL }}/{{ Campus.ShortCode }}/{{ PageParameter.Theme }}" onclick="return saveLink(this);">{{ Campus.Name }}</a>
            {% endif %}
        {% endfor %}
    
        <br />
        
        <h4>OR</h4>
        
        <h2>Switch Themes</h2>
        {% assign Themes = ThemeList | Split:',' %}
        {% for Theme in Themes %}
            <a class="autoconfigbutton" href="{{ baseURL }}{{ AutoConfigURL }}?Theme={{ Theme }}">
                {{ Theme | Humanize | Capitalize | Replace:'New Pointe','NewPointe' }}
            </a>
        {% endfor %}
    </div>
{% endif %}

```

4. Refresh the page and test it!
