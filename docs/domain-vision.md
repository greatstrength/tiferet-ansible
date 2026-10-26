# Domain Vision Statement - Tiferet Ansible

**Status:** Draft (pre-implementation) - **Domain:** `tiferet-ansible` - **Code:** none yet - **Branch:** `docs-core-domain` (planned)

> Pre-implementation note: the tiferet-ansible repository currently holds only a README, a license and a .gitignore. This statement describes the intended domain and its value, not existing code.

## The bet: manage the model, render the YAML

Most teams that automate servers with Ansible keep their whole operation in hand-edited YAML files: an inventory that lists the machines, scattered variable files that say how to reach each one, and playbooks that say what to do to them. Those files work, but they are the only record of the setup. Nothing checks that a playbook is correctly written, that a host's connection settings are complete, or that two people editing the same file have not quietly broken each other's changes. Mistakes surface when a run fails against real machines.

Tiferet Ansible takes the opposite bet. The machines, how each one is reached, and the playbooks that act on them are kept as structured records that the application owns and checks. Ansible's YAML files are produced from those records when they are needed, in one direction only. The YAML is output, like a compiled file, not the place where the truth lives. Change the record and the next render reflects it; nobody edits the generated YAML by hand.

## What this domain makes real

Tiferet Ansible is a management layer, built on the Tiferet framework, that holds an organization's Ansible inventory, host connection settings and playbooks as checked records. Through one interface, an operator or another program can manage inventories, hosts, connection settings and playbooks, and, as a separate step, render the matching Ansible files into an output project folder that each playbook can set for itself. The first release stops there: it manages and renders. Running playbooks against machines comes later.

## What we get for it

**Fewer failed runs against live machines.** A change that is not correctly written is rejected when the change is made, with a named error, instead of being discovered halfway through a deployment. For example, a play whose list of target hosts uses a pattern Ansible cannot read would be refused at the point of the edit. Every generated file is also checked as valid Ansible before it is handed on.

**One place to answer "how do we reach that server?"** Each host's connection details (address, port, login user, connection type, any jump or bastion host it must go through, and related settings) live on that host's record. Today that answer is often spread across several files; here it has one home.

**Changes that other tools can make safely.** Because every change goes through the same set of named operations (add a host, assign it to a group, add a task to a play), a script, a command line or a future web interface can all edit the setup the same way and get the same checks. No caller needs to know how the YAML is laid out.

**Generated files that stay consistent.** Two renders of the same records produce the same files, so reviews show real changes, not formatting noise. Ansible settings the records do not yet model are carried through unchanged.

**Secrets kept out of the records.** Passwords and keys are referenced, not stored, and encrypted vault values are kept as-is, never decrypted.

## The core of the work

The domain follows a short path:

> **Declare** inventories, hosts, connection settings and playbooks -> **check** each change as it is made -> **store** the records -> **render** them on request into standard Ansible YAML -> **check** that output as valid Ansible.

In practice the workflow is: add and edit records freely, then render to seal the deal. Saving a change checks it and keeps it with the records, but writes no Ansible file. Rendering is the deliberate step that writes the current records out as Ansible files in the output project folder.

The central design commitment is **one-way rendering: the records are the source of truth and Ansible YAML is always derived from them.** This part works like a compiler: it checks that what it is given is correctly written, not whether the machines it names exist or are running. The link between a playbook and an inventory is kept by name, the way Ansible itself does it, so the same playbook can be rendered against a test inventory or a production one without being rewritten.

## What it deliberately does not do

It does not run playbooks in the first release. Executing a rendered playbook against real machines is planned later scope, as is the services management named in the repository's tagline. Until then, rendered files go to Ansible through the team's existing process.

It does not replace Ansible. Ansible still owns connecting to machines, running modules, applying its variable precedence rules and interpreting template expressions. Tiferet Ansible produces the files Ansible reads; it does not reimplement what Ansible does with them.

It does not provide its own framework. Request handling, error catalogs, configuration storage and dependency wiring come from Tiferet; checking of small languages such as host patterns comes from Tiferet's language package. This domain supplies only the Ansible-specific records, rules and rendering on top.

It does not check the live world. It never contacts machines or asks whether a named host is up; a running host is its own service, outside this package.

It does not decrypt or store secrets. Vault passwords, SSH keys and privilege-escalation passwords stay with the team's existing secret handling and are referenced by environment variable or file path.

---

*Companion document:* `docs/core-domain-distillation.md` - the detailed walkthrough of the domain's vocabulary, behaviors, and the relationships between its parts (drafted after this statement).
