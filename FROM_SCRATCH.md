# CommCare Ansible Monolith README

## commcarehq-ansible
1. Create Ubuntu 14.04 box with your public SSH key for root access.
2. Clone this repository.
3. Create and add the public and private SSH keys you will use for vagrant to the `/provisioning` folder as `id_rsa` and `id_rsa.pub`.
4. Add your public key file to `/ansible/vars/users` as `myname.pub` and add `myname` to the list of active users in `ansible/group_vars/all.yml` and `ansibl/vars/dev_public.yml`.
5. Edit `ansible/inventories/monolith` to include your box's IP.
6. `vagrant up`
7. `vagrant ssh control`
8. `ansible-playbook -i inventories/monolith -e '@vars/dev/dev_private.yml' -e '@vars/dev/dev_public.yml' deploy_stack.yml -u root --tags=users`
9. `ssh` onto target-box as root and set the ansible user's password with `passwd ansible`.
10. `ansible-playbook -i inventories/monolith -e '@vars/dev/dev_private.yml' -e '@vars/dev/dev_public.yml' deploy_stack.yml -u ansible --ask-become-pass` and enter the new password when prompted.
11. re-run `ansible-playbook -i inventories/monolith -e '@vars/dev/dev_private.yml' -e '@vars/dev/dev_public.yml' deploy_stack.yml -u ansible --ask-become-pass`
12. `ssh` onto target-box and
  1. `su cchq`
  2. `cd ~/www/dev/current`
  3. `source python_env/bin/activate`
  4. `./manage.py sync_couch_views`
  5. `env CCHQ_IS_FRESH_INSTALL=1 ./manage.py migrate --noinput`
  6. `ctrl-d`
  7. `ctrl-d`
13. `ansible-playbook -i inventories/monolith -e '@vars/dev/dev_private.yml' -e '@vars/dev/dev_public.yml' deploy_stack.yml -u ansible --ask-become-pass --start-at-task='Touchforms user'`

### N.B.: If you ever reboot the target-box, you must run:
`ansible-playbook -i inventories/monolith -e '@vars/dev/dev_private.yml' -e '@vars/dev/dev_public.yml' deploy_stack.yml -u ansible --ask-become-pass --tags='after-reboot'`

## commcare-hq-deploy
1. cd `~/commcarehq-ansible/commcare-hq-ansible`
1. Setup `fab/environments.yml`, `fab/fabfile.py`, and `/inventory/dev` for you `dev` box.
2. `~/commcare-hq-deploy/python_env/bin/fab dev deploy -u root` fails with `OSError: [Errno 13] Permission denied: '/home/cchq/www/dev/releases/2017-10-30_17.15/python_env/lib/python2.7/site-packages/pytz-2015.7.dist-info/DESCRIPTION.rst'`
4. `ssh root@<target-box-ip>`
  1. `cd ~cchq/www/dev/current && chown -R cchq.cchq python_env`
  2. `ctrl-d`
5. `./python_env/bin/fab dev deploy -u root`
6. `./python_env/bin/fab dev deploy:resume=yes -u root`
