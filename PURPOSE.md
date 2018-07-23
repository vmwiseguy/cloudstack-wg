**Open Mainframe Project**

Following is the vision and mission of the Open Mainframe Project (OMP).

**Vision**

Linux on the Mainframe as the standard for enterprise class systems and applications

**Mission**

Build community and adoption of Open Source on the mainframe by:

* **Eliminating barriers** to Open Source adoption on the mainframe

* **Demonstrating value** of the mainframe on technical and business levels

* **Strengthening collaboration** and resources for the community to thrive

**The "Cloudstack" working group**

There is a working group that meets every other Friday from 11:00-12:00 Eastern time.  How can this group be more effective in attaining the the OMP mission?

What is the goal of this working group/subcommittee?

* Maintain this document on how to deploy an open source based private cloud on System z

* Create a 3-hour class for SHARE/VM Workshop/etc. to show Private cloud on z?

**Private cloud solution high level goals**

Following are ten high level goals that have been considered in the design and construction of a Private cloud solution on System z

1.    Meet users' real needs - the solution should be acceptable to end users, not administrators.

2.    Use a phased approach - *Continuous delivery* should work for this solution.

3.    Ensure accessibility - The self-service portal should work for all browsers and be easy to navigate.

4.    Ensure usability - Elementary tasks should be easy and drill-down intuitive.

5.    Personalize the portal - allow user preferences to be set.

6.    Keep content clear and concise - Do not require deep IT knowledge nor platform specifics.

7.    Offer multiple means of help - Documents, articles hover-help, links to chat, defect opening, etc.

8.    Measure the portal - Constantly monitor the portal's performance.

9.    Continuous improvement - Get user feedback to improve usability and performance

10. Availability - The portal should never go down.

**Private cloud specific goals**

Following are more specific goals of a Private cloud solution on z:

Allow *self-service* - Users with credentials will be able to build, rebuild, destroy, configure and report on zLinux guests for the groups that they are members of.

Require *authentication* - Users must first authenticate with valid Linux user/password credentials before they can access the self-service portal.

Enforce *authorization* - Users will only be able to operate on systems in their Linux group. Those who are members of more than group will be able to change groups, in order to operate on guests in that group. Members of the vmlinux group will be treated as administrators and will be able to see all groups.

Enable *groups* - the main grouping guests will be based on users’ primary Linux group. Within each group, a secondary grouping mechanism is afforded. This grouping level is named *projects*.

Allocate *quotas* - groups are given quotas for the number of CPUs and the amount of memory. Once a group has exhausted their quota of CPUs or memory, members of that group will not be able to provision more guests. Administrators will be able to set groups’ quotas.

Allow guest *filtering* - to manage the list of guests displayed in the portal, associates will be easily able to filter the list of guests by host name, or by project. Administrators will also be able to filter on group.

Maintain an *audit trail* - all operations must be logged with user names and the groups they are running as.

Enable additional function for System z administrators, including the ability to modify quotas, permanently delete guests, run arbitrary commands

**Required operations**

The following operations are required for self-service.

Operations related to **security** are as follow:

* **Login                    	**Gain access to the self-service environment

* **Logout                 	**End the self-service session

Operations related to **cloning** guests are as follow:

* **Build                     	**Create new guest(s) in new virtual machine(s)

* **Rebuild                	**Replace Linux OS and data in existing virtual machine(s)

* **Disable		**Disable guest; not usable/startable (resources remain)

* **Purge                   	**Actually delete a destroyed system (admins only)

Operations related to **recycling **guests are as follow:

* **Power on            	**Boot powered-down guest

* **Power off           	**Shut down running guests (soft/hard down)

* **Reboot (hard)   	**Shut down, log off, log on then boot guests

* **Reboot (soft)     	**Recycle guests without a z/VM logoff

* **Stop                     	**Freeze all CPUs (administrators only)  maybe not

* **Start                    	**Resume frozen guests (administrators only)   maybe not

Operations to **configuring** guests are as follow. These operations will correspondingly add to or detract from the group’s quotas.

JSV NOTES:  Add/Remove CPU and MEMORY could easily be ONE API each.  For instance, a call with "Modify_CPU" and CPUs=n or CPUs=+n or CPUS=-n.  Without +/- the cloud-host would deterime if it is adding or removing CPUs

* **Add CPUs            	**Dynamically add CPUs to guests up to a set maximum

* **Remove CPUs    	**Dynamically remove CPUs from guests down to one

* **Add memory      	**Dynamically add memory to running guests

* **Remove memory	**Dynamically remove memory from running guests

* **Capping/SHARE	**Dynamically change SHARE settings (capping, etc)

* **Add/expand disk	**Minidisks/EDEV/DASD

* **Set group        	**Change the group running guests (for scope of view/control)

Operations to **Reporting** on guests are as follow:

* **System details        **Report on all aspects of selected systems

* **System expirations	**Report on when selected systems expire

* **Capacity reporting 	**Point to ES Capacity Planning guest

* **LIST			**Simple listing of all existing guests

 

 

**API Recommendations**

**Common call structure for all API calls**

Through a secure/SSL POST call to the Cloud Host, a JSON data block (body) would be constructed and sent containing the API name in the header as a dict type and any required data for the API.

*	{"api_name": {“var1”:”val1”,“var2”:”val2”, … } }*

**Common Responses to all API calls**

The header of the JSON data should contain the following for all calls:

	

<table>
  <tr>
    <td>rc</td>
    <td>The return code for the API called</td>
  </tr>
  <tr>
    <td>rs</td>
    <td>The reason code for the API called</td>
  </tr>
  <tr>
    <td>apimsg</td>
    <td>Any error or status message for the API call</td>
  </tr>
  <tr>
    <td>errfunc</td>
    <td>The function or process that caught the error</td>
  </tr>
</table>


**Proposed API List**

The following is a proposed list of APIs, along with input, output and any other notes of interest for each

<table>
<tbody>
<tr>
  <td>API</td>
  <td>Input</td>
  <td>Output</td>
  <td>Other Info</td>
</tr>
<tr>
  <td>version</td>
  <td>Default - no API</td>
  <td>Version details of cloud manager</td>
  <td>No authority required</td>
</tr>
<tr>
  <td>Sign-in (get token)</td>
  <td>Admin-token-key in header, with &ldquo;API host key&rdquo; along with uid/pwd for authentication</td>
  <td>Authorized token string</td>
  <td>Additional auth calls can be done with token and no uid/pwd</td>
</tr>
<tr>
  <td>Sign-out (invalidate token)</td>
  <td>Authorized token key</td>
  <td>Basic results block (return code)</td>
  <td>Allows the token to be invalidate so it can&rsquo;t be used again</td>
</tr>
<tr>
  <td>List Guests</td>
  <td><ul>
    <li>Nothing</li>
    <li>A &ldquo;host&rdquo; name (LPAR) ?</li>
    <li>By GROUP ?</li>
    </ul></td>
  <td>Json array of guest names; possibly include NODE or HOST names (LPAR)?</td>
  <td>Do people NEED to see guests on a specific HOST or should there be a HOST (LPAR) included in the list for Enterprise views?</td>
</tr>
<tr>
  <td>List Images</td>
  <td>Nothing</td>
  <td>Json array of image names and their characteristics; OS type/level, description, etc</td>
  <td>Stored images for cloning new servers. Could be a kickstart &ldquo;process&rdquo; also.</td>
</tr>
<tr>
  <td>Create Guest</td>
  <td><ul>
    <li>New userid</li>
    <li>Clone-from image name (userid) (optional?)</li>
    <li>Overrides:</li>
    </ul>
    <ul>
    <li>CPUs</li>
    <li>MEM</li>
    <li>IP, VSWITCH, VLAN (etc)</li>
    <li>others...</li>
    </ul></td>
  <td>Basic results block (return code)</td>
  <td></td>
</tr>
<tr>
  <td>Add disk</td>
  <td>Userid, lists of disks (vdev, size, format (or source uid/disk), diskpool)</td>
  <td>Basic results block (return code)</td>
  <td></td>
</tr>
<tr>
  <td>Delete disk</td>
  <td>Userid and list of vdevs to delete</td>
  <td>Basic results block (return code)</td>
  <td></td>
</tr>
<tr>
  <td>Guest Details</td>
  <td>userid</td>
  <td>Json block including cpu, memory, disks/vdevs/size, etc.</td>
  <td>Basically the directory details but formatted for json</td>
</tr>
<tr>
  <td>Guest Directory</td>
  <td>userid</td>
  <td>user&rsquo;s directory entry</td>
  <td>Pwds masked</td>
</tr>
<tr>
  <td>Delete Guest</td>
  <td><ul>
    <li>Userid</li>
    <li>How: Purge or Disable (optional; default action defined on manager)</li>
    </ul></td>
  <td>Basic results block (return code)</td>
  <td></td>
</tr>
<tr>
  <td>Guest Status</td>
  <td>userid</td>
  <td>Monitor details&hellip;.???</td>
  <td></td>
</tr>
<tr>
  <td>Start guest</td>
  <td>userid</td>
  <td>Basic results block (return code)</td>
  <td></td>
</tr>
<tr>
  <td>Stop guest</td>
  <td><ul>
    <li>Userid</li>
    <li>How: (hard or soft)</li>
    </ul></td>
  <td>Basic results block (return code)</td>
  <td></td>
</tr>
<tr>
  <td>Pause/Unpause guest</td>
  <td></td>
  <td></td>
  <td>???? value ????</td>
</tr>
<tr>
  <td>Reboot guest</td>
  <td><ul>
    <li>Userid</li>
    <li>How: hard or soft; hard means re-IPL without shutdown attempt; soft is default</li>
    </ul></td>
  <td>Basic results block (return code)</td>
  <td>Graceful shutdown and restart</td>
</tr>
<tr>
  <td>Reset guest</td>
  <td><ul>
    <li>Userid</li>
    <li>How: hard or soft; hard means LOGOFF without shutdown attempt; soft is default</li>
    </ul></td>
  <td>Basic results block (return code)</td>
  <td>Graceful shutdown, logoff VM and AUTOLOG</td>
</tr>
<tr>
  <td>Guest Console</td>
  <td><ul>
    <li>Userid</li>
    <li>Other parms to limit output?</li>
    </ul></td>
  <td>The console data</td>
  <td>This could be unwieldy if not limited; will required setting up on VM side</td>
</tr>
<tr>
  <td>Deploy (clone??) guest</td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>Guest Powerstate</td>
  <td>userid</td>
  <td>On or Off</td>
  <td>Is userid running or not?</td>
</tr>
<tr>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>SYSTEM APIs - like VSWITCH, etc??</td>
  <td></td>
  <td></td>
  <td></td>
</tr>
</tbody>
</table>
