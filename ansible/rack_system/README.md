

Copy the `site_vars_example.yml` file and fill in with variable values for the site being installed. 
`cp site_vars_example.yml site_vars.yml` Replace all placeholder values in angle brackets,


Run the playbook using `ansible-playbook -i hosts playbook.yml`
Add `--check` to run without actually making changes on remote host.  
Add `--ask-pass` if password needed to login to node.  
Add `--ask-become-pass` if sudo password needed on node.  
