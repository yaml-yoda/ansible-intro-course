# Intro to Ansible - Simple Project

This project walks you through basic Ansible commands and playbooks, then extends into API integrations.

Learning flow:

1. Start with inventory, ad-hoc commands, and a simple local playbook.
2. Fetch data from a public Star Wars API and parse JSON results.
3. Use a public NetBox demo instance with an API token to create objects from that data.

This project is a minimal starter for learning:

- inventory
- ad-hoc command
- playbook execution
- variables
- idempotency
- API calls and JSON parsing
- passing input through variables (`vars` and `-e`)

## Services Integrated

- Star Wars API (SWAPI): [`https://swapi.py4e.com/api/`](https://swapi.py4e.com/api/)
  - Public, no-auth API used to fetch character and film data for demos.
- Public NetBox demo: [`https://demo.netbox.dev`](https://demo.netbox.dev)
  - Used to practice NetBox API integration with token-based authentication and object creation.

## Prerequisites

- Python 3
- Ansible installed (`ansible` and `ansible-playbook` commands available)

## Project Structure

```text
ansible-intro-course/
├── ansible.cfg
├── requirements.yml
├── group_vars/
│   └── all.yml
├── inventories/
│   └── lab/
│       └── hosts.ini
└── playbooks/
    ├── 01_site_basics.yml
    ├── 02_api_star_wars.yml
    ├── 03_api_films_to_student_file.yml
    ├── 04_netbox_create_contact_from_starwars.yml
    └── 05_netbox_module_contact_from_starwars.yml
```

## 1) Validate setup

From this folder:

```bash
cd ansible-intro-course
ansible --version
```

## 2) Test inventory with an ad-hoc command

```bash
ansible all -m ping
```

You should get a `pong` response from `student-node`.

## 3) Run the first playbook (basic modules)

```bash
ansible-playbook playbooks/01_site_basics.yml
```

What this playbook does:

1. Pings the host
2. Creates a simple note file in `/tmp`
3. Shows a custom message variable

## 4) Run it again (idempotency check)

Run the same command a second time:

```bash
ansible-playbook playbooks/01_site_basics.yml
```

Expected: fewer or no changed tasks on the second run.

## 5) Run the second playbook (public API + parsing)

```bash
ansible-playbook playbooks/02_api_star_wars.yml
```

What this playbook does:

1. Calls a public Star Wars API (`https://swapi.py4e.com`) with no auth
2. Reads JSON fields from the response
3. Prints parsed values (name, birth year, height, film count)

## 6) Run the third playbook (API + write student file)

```bash
ansible-playbook playbooks/03_api_films_to_student_file.yml
```

What this playbook does:

1. Fetches one character from SWAPI
2. Fetches each film URL for that character
3. Writes all film titles to `note_file_path` (default: `/tmp/ansible_intro_note.txt`)

## 7) Run the fourth playbook (input var + create in NetBox)

Set your NetBox token first:

```bash
export NETBOX_TOKEN='your_token_here'
```

Run with default input (`Luke Skywalker`):

```bash
ansible-playbook playbooks/04_netbox_create_contact_from_starwars.yml
```

Run with custom input using `-e` (extra vars):

```bash
ansible-playbook playbooks/04_netbox_create_contact_from_starwars.yml -e 'sw_character="Leia Organa"'
```

What this playbook does:

1. Takes a Star Wars character as input variable (`sw_character`)
2. Fetches the person from SWAPI (`/people/?search=...`)
3. Creates that character as a contact in NetBox (`/api/tenancy/contacts/`)

## How variable input works in these demos

1. `vars` inside a playbook set default values.
2. `lookup('env', 'NETBOX_TOKEN')` reads input from environment variables.
3. `-e "key=value"` overrides a variable at run time (useful for student input).

## 8) Run the fifth playbook (NetBox Ansible module)

Install the NetBox collection and Python dependency:

```bash
ansible-galaxy collection install -r requirements.yml
python3 -m pip install pynetbox
```

Run with default input:

```bash
ansible-playbook playbooks/05_netbox_module_contact_from_starwars.yml
```

Run with custom input:

```bash
ansible-playbook playbooks/05_netbox_module_contact_from_starwars.yml -e 'sw_character="Leia Organa"'
```

What this playbook does:

1. Takes `sw_character` as input (from `vars` default or `-e` override)
2. Fetches the person from SWAPI
3. Uses `netbox.netbox.netbox_contact` to ensure the contact exists in NetBox
4. Uses a deterministic email as the idempotent lookup key (avoids duplicate-name conflicts)

Troubleshooting (show full module logs):

```bash
ansible-playbook playbooks/05_netbox_module_contact_from_starwars.yml -e debug_module_output=true
```

## Suggested Exercises

1. Change `course_message` in `group_vars/all.yml`.
2. In `02_api_star_wars.yml`, change `swapi_person_id` from `1` to `5` and compare output.
3. Add one more parsed field (for example `eye_color` or `mass`) and print it.
