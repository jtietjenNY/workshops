# Exercise 2.7 - Wrap up

# Final Challenge or Putting it all Together

This is the final challenge where we try to put most of what you have learned together.

## Let’s set the stage

Your operations team and your application development team like what they see in Tower. To really use it in their environment they put together these requirements:

- All webservers (`node1`, `node2` and `node3`) should go in one group

- As the webservers can be used for development purposes or in production, there has to be a way to flag them accordingly as "stage dev" or "stage prod".

    - Currently `host1` is used as a development system and `host2` is in production.

- Of course the content of the world famous application "index.html" will be different between dev and prod stages.

    - There should be a title on the page stating the environment

    - There should be a content field

- The content writer `wweb` should have access to a survey to change the content for dev and prod servers.

## The Git Repository

All code is already in place - this is a Tower lab after all. Check out the **Ansible Workshop Examples** git repository at **https://github.com/ansible/workshop-examples**. There you will find the playbook `webcontent.yml`, which calls the role `role_webcontent`.

Compared to the previous Apache installation role there is a major difference: there are now two versions of an `index.html` template, and a task deploying the template file which has a variable as part of the source file name:

`dev_index.html.j2`

```html
<body>
<h1>This is a development webserver, have fun!</h1>
{{ dev_content }}
</body>
```

`prod_index.html.j2`

```html
<body>
<h1>This is a production webserver, take care!</h1>
{{ prod_content }}
</body>
```

`main.yml`

```yaml
[...]
- name: Deploy index.html from template
  template:
    src: "{{ stage }}_index.html.j2"
    dest: /var/www/html/index.html
  notify: apache-restart
```

## Prepare Inventory

There is of course more then one way to accomplish this, but here is what you should do:

- Make sure all hosts are in the inventory group `Webserver`.

- Define a variable `stage` with the value `dev` for the `Webserver` inventory:

    - Add `stage: dev` to the inventory `Webserver` by putting it into the **VARIABLES** field beneath the three start-yaml dashes.

- In the same way add a variable `stage: prod` but this time only for `node2` (by clicking the hostname) so that it overrides the inventory variable.

> **Tip**
>
> Make sure to keep the three dashes that mark the YAML start\!

## Create the Template

- Create a new **Job Template** named `Create Web Content` that

    - targets the `Webserver` inventory

    - uses the Playbook `rhel/apache/webcontent.yml` from the new **Ansible Workshop Examples** Project

    - Defines two variables: `dev_content: default dev content` and `prod_content: default prod content` in the **EXTRA VARIABLES FIELD**

    - Uses `Workshop Credentials` and runs with privilege escalation

- Save and run the template

## Check the results

This time we use the power of Ansible to check the results: execute curl on each node locally, orchestrated by an ad-hoc command on the command line:

```bash
$ ansible web -m command -a "curl -s http://localhost:80"
 [WARNING]: Consider using the get_url or uri module rather than running 'curl'.  If you need to use command because get_url or uri is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.

node2 | CHANGED | rc=0 >>
<body>
<h1>This is a production webserver, take care!</h1>
prod wweb
</body>

node1 | CHANGED | rc=0 >>
<body>
<h1>This is a development webserver, have fun!</h1>
dev wweb
</body>

node3 | CHANGED | rc=0 >>
<body>
<h1>This is a development webserver, have fun!</h1>
dev wweb
</body>
```

Note the warining in the first line about not to use `curl` via the `command` module since there are better modules right within Ansible. We will come back to that in the next part.

## Add Survey

- Add a survey to the Template to allow changing the variables `dev_content` and `prod_content`

- Add permissions to the Team `Web Content` so the Template **Create Web Content** can be executed by `wweb`.

- Run the survey as user `wweb`

Check the results. Since we got a warning last time using `curl` via the `command` module, this time we will use the dedicated `uri` module. As arguments it needs the actual URL and a flag to output the body in the results.

```bash
$ ansible web -m uri -a "url=http://localhost return_content=yes"
node2 | SUCCESS => {
    "accept_ranges": "bytes",
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "connection": "close",
    "content": "<body>\n<h1>This is a production webserver, take care!</h1>\nprod wweb\n</body>\n",
    "content_length": "77",
    "content_type": "text/html; charset=UTF-8",
    "cookies": {},
    "cookies_string": "",
    "date": "Wed, 10 Jul 2019 22:15:45 GMT",
    "elapsed": 0,
    "etag": "\"4d-58d5aef2a5666\"",
    "last_modified": "Wed, 10 Jul 2019 22:09:42 GMT",
    "msg": "OK (77 bytes)",
    "redirected": false,
    "server": "Apache/2.4.6 (Red Hat Enterprise Linux)",
    "status": 200,
    "url": "http://localhost"
}
[...]
```

## Solution

> **Warning**
>
> **Solution Not Below**

You have done all the required configuration steps in the lab already. If unsure, just refer back to the respective chapters.

# The End

Congratulations, you finished your labs\! We hope you enjoyed your first encounter with Ansible Tower as much as we enjoyed creating the labs.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-2---ansible-tower-exercises)
