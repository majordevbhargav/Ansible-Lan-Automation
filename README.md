# Network Device Command Runner - Ansible version

Same idea as the Python script, built as an Ansible playbook instead:
run commands (e.g. `show run`) across Cisco and HP Aruba switches, and
save the output to files. No Python code to write or maintain -
everything below is YAML + a couple of CLI commands.

Note: Ansible itself is written in Python and needs Python installed
on the machine you run it *from* (not on the switches). If your goal
was "nothing Python-related anywhere," see the Bash+SSH option from
earlier instead. If your goal was "I don't want to write/maintain
Python code," this fits.

## 1. Install Ansible and the collections

```bash
pip install ansible-core
ansible-galaxy collection install -r requirements.yml
```

`requirements.yml` pulls in:
- `cisco.ios` - Cisco IOS/IOS-XE support
- `arubanetworks.aos_switch` - ArubaOS-Switch (HP Aruba) support
- `ansible.netcommon` - shared connection plugin both rely on

## 2. Fill in and encrypt your credentials

Edit `group_vars/all/vault.yml` and replace the placeholders:

```yaml
vault_net_user: "ram"
vault_net_pass: "your-real-password"
```

Then encrypt the file so the password never sits on disk in plain
text:

```bash
ansible-vault encrypt group_vars/all/vault.yml
```

You'll be asked to set a vault password - remember it, you'll need
it every time you run the playbook. If you ever need to edit the
credentials again, use `ansible-vault edit group_vars/all/vault.yml`
rather than opening the file directly.

## 3. Edit the inventory / commands

- `inventory.yml` already lists your two switches (Cisco at
  10.5.1.20, Aruba at 172.29.129.63) - add more under the matching
  group as needed.
- `commands.yml` lists the commands to run on every device - add or
  remove lines there.

## 4. Run it

```bash
ansible-playbook -i inventory.yml playbook.yml -e @commands.yml --ask-vault-pass
```

You'll be prompted for the vault password you set in step 2 (not
the switch password - that's already stored, encrypted, in the
vault file).

Only target one device:

```bash
ansible-playbook -i inventory.yml playbook.yml -e @commands.yml --ask-vault-pass --limit core-switch-01
```

## Output

Each device gets its own timestamped file under `./output/`, e.g.
`output/core-switch-01_20260901_143500.txt` - same layout as the
Python version, easy to open in any text editor.

## Ideas for next steps

- **Config changes, not just show commands**: `cisco.ios.ios_config`
  and the equivalent Aruba module can push configuration the same
  way this pulls output - handy for standardizing settings across
  the fleet.
- **Scheduled runs**: wire this playbook into cron (Linux/Mac) or
  Task Scheduler (Windows) for a nightly "pull show run from every
  switch" job, giving you a running history of configs.
- **Diffing over time**: since every run is timestamped, diffing two
  output files for the same device shows exactly what changed
  between runs - useful for change control.
- **Enable/privileged mode**: if any commands need enable mode on
  Cisco, add an `ansible_become: true` / `ansible_become_method:
  enable` pair to that device's vars, with the enable secret also
  pulled from the vault.
- **Bring in the Aruba posture / ISE work**: since you're also
  building the ISE External Posture Assessment POC, this same
  Ansible pattern could later trigger posture checks or pull
  interface/MAC data as part of that compliance flow, if useful.

## Troubleshooting

- **"couldn't resolve module/action" for the Aruba task**: make sure
  you're on the latest version of this playbook - the Aruba module
  is `arubanetworks.aos_switch.arubaoss_command`, not `aos_command`.
  Aruba actually ships two separate collections: `aos_switch` for
  older ArubaOS-Switch gear (what this playbook targets) and
  `aoscx` for newer AOS-CX switches. If your switch is AOS-CX, let
  me know and the playbook needs different module names
  (`aoscx_command`).
- **Aruba task fails with a "network_os" or "group_modules" error**:
  try re-running with this environment variable set first:
  `export ANSIBLE_NETWORK_GROUP_MODULES=arubaoss` (Linux/WSL), then
  run the playbook again in the same terminal session.

## CSV export (for Excel)

Every run now also writes `output/switch_report_<timestamp>.csv`,
a single file covering every device and every command, with columns:
`device, host, command, timestamp, output`.

Double-click it and Excel will open it directly. Multi-line output
(like a full `show run`) sits correctly inside one cell, wrapped in
quotes - just widen the row or turn on "Wrap Text" in Excel to read
it comfortably. If double-clicking ever shows garbled characters
instead of opening cleanly, use Excel's **Data > From Text/CSV**
import option instead, which lets you confirm UTF-8 encoding.

The per-device `.txt` files are still written too, in case you want
raw text for a specific device rather than the combined view.

## Security note

Same as before: never put real credentials directly into
`inventory.yml`, `playbook.yml`, or any file that isn't run through
`ansible-vault encrypt`. The vault file is the only place the
password should live, and only in its encrypted form.