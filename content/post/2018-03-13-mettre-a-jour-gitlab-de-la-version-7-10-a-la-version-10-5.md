---
title: "Mettre à jour GitLab CE Omnibus de la version 7.10 à la version 10.5"
date: 2018-03-12T15:34:10+01:00
draft: false
author: VaLouille
type: post
categories:
  - GitLab
tags:
  - gitlab
  - gitlab-ci
  - upgrade
  - omnibus

---

GitLab Omnibus est la version la plus simple à installer de GitLab. GitLab Omnibus est disponible depuis la version 6.6 et se présente sous la forme d'un unique paquet `.deb` ou `.rpm`. La mise à jour se fait comme pour n'importe quel paquet, mais faire un trop gros saut de version en une seule fois risque de ne pas fonctionner.

> Pour les versions antérieures à 7.10, il est possible de la mettre à jour en 7.10 au préalable en suivant [cette documentation][1]

J'ai déjà réalisé cette mise à jour en suivant le chemin de mise à jour suivant :

  * 7.10 -> 8.0.5
  * 8.0.5 -> 8.5.13
  * 8.5.13 -> 8.9.5
  * 8.9.5 -> 8.11.9
  * 8.11.9 -> 8.17.8
  * 8.17.8 -> 9.0.13
  * 9.0.13 -> 9.5.9
  * 9.5.9 -> 10.1.7
  * 10.1.7 -> 10.5.4

Il est certainement possible d'effectuer des sauts de version plus importants, mais j'ai preféré y aller petit à petit. (La 8.11 induit des [changements au niveau des secrets][4], la 9.5.9 [met à jour PostgreSQL vers la version 9.6][5])

> Évidemment, il est préférable d'installer les mises à jour de GitLab au fur et à mesure de leur parution, le processus étant simple et fonctionnant bien.

Avant tout chose, on crée une sauvegarde de GitLab :

```
~ # sudo gitlab-rake gitlab:backup:create
Dumping database ...
Dumping PostgreSQL database gitlabhq_production ... [DONE]
done
Dumping repositories ...
[...]
done
Dumping uploads ...
done
Creating backup archive: 1620838995_gitlab_backup.tar ... done
Uploading backup archive to remote storage  ... skipped
Deleting tmp directories ... done
done
done
done
Deleting old backups ... skipping
```

Les paquets peuvent être téléchargés à partir de [l'adresse suivante][3]. Il est possible de rechercher les versions, et filtrer par version d'OS.

Pour chaque version, on la télécharge, et on installe le paquet :

```
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/debian/wheezy/gitlab-ce_8.0.5-ce.0_amd64.deb/download.deb
```

```
dpkg -i gitlab-ce_8.0.5-ce.0_amd64.deb
```

L'output devrait ressembler à ça :
```
# dpkg -i gitlab-ce_8.0.5-ce.0_amd64.deb
(Reading database ... 114240 files and directories currently installed.)
Preparing to replace gitlab-ce 7.10.1~omnibus-1 (using .../gitlab-ce_8.0.5-ce.0_amd64.deb) ...
gitlab preinstall: Backing up GitLab SQL database (excluding Git repositories, uploads)
Dumping database ...
Dumping PostgreSQL database gitlabhq_production ... [DONE]
done
Dumping repositories ...
[SKIPPED]
Dumping uploads ...
[SKIPPED]
Creating backup archive: 1520849200_gitlab_backup.tar ... done
Uploading backup archive to remote storage  ... skipped
Deleting tmp directories ... done
done
Deleting old backups ... skipping
Unpacking replacement gitlab-ce ...
Setting up gitlab-ce (8.0.5-ce.0) ...
gitlab: Thank you for installing GitLab!
gitlab: To configure and start GitLab, RUN THE FOLLOWING COMMAND:

sudo gitlab-ctl reconfigure

gitlab: GitLab should be reachable at http://gitlab.domaine.com/
gitlab: Otherwise configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
gitlab: And running reconfigure again.
gitlab:
gitlab: For a comprehensive list of configuration options please see the Omnibus GitLab readme
gitlab: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md
gitlab:
Shutting down all GitLab services except those needed for migrations
ok: down: logrotate: 0s, normally up
ok: down: nginx: 1s, normally up
ok: down: postgresql: 0s, normally up
ok: down: redis: 0s, normally up
ok: down: sidekiq: 0s, normally up
ok: down: unicorn: 0s, normally up
ok: run: postgresql: (pid 20927) 0s
ok: run: redis: (pid 20935) 1s
run: postgresql: (pid 20927) 1s; run: log: (pid 16283) 11816231s
run: redis: (pid 20935) 1s; run: log: (pid 16278) 11816231s
Reconfiguring GitLab to apply migrations
Starting Chef Client, version 12.4.1
resolving cookbooks for run list: ["gitlab"]
Synchronizing Cookbooks:
  - gitlab
  - package
  - runit
Compiling Cookbooks...
Recipe: gitlab::default
  * directory[/etc/gitlab] action create (up to date)
[2018-03-12T11:07:42+01:00] WARN: Cloning resource attributes for directory[/var/opt/gitlab] from prior resource (CHEF-3694)
[2018-03-12T11:07:42+01:00] WARN: Previous directory[/var/opt/gitlab]: /opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/recipes/default.rb:43:in `from_file'
[2018-03-12T11:07:42+01:00] WARN: Current  directory[/var/opt/gitlab]: /opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/recipes/users.rb:24:in `from_file'
[2018-03-12T11:07:42+01:00] WARN: Cloning resource
[...]
Recipe: gitlab::postgresql
  * service[postgresql] action restart
    - restart service service[postgresql]
Recipe: gitlab::gitlab-git-http-server
  * ruby_block[reload gitlab-git-http-server svlogd configuration] action create
    - execute the ruby block reload gitlab-git-http-server svlogd configuration

Running handlers:
Running handlers complete
Chef Client finished, 52/201 resources updated in 26.458720146 seconds
gitlab Reconfigured!
Restarting previously running GitLab services
ok: run: logrotate: (pid 21486) 0s
ok: run: nginx: (pid 21492) 1s
ok: run: postgresql: (pid 21466) 3s
ok: run: redis: (pid 20935) 31s
ok: run: sidekiq: (pid 21506) 0s
ok: run: unicorn: (pid 21510) 0s

Upgrade complete! If your GitLab server is misbehaving try running

   sudo gitlab-ctl restart

before anything else. If you need to roll back to the previous version you can
use the database backup made during the upgrade (scroll up for the filename).
```

Il faut réitérer les opérations ci-dessus pour chaque saut de version.

Une fois toutes les mises à jour appliquées, il peut être nécessaire de recharger la configuration via la commande `gitlab-ctl reconfigure`. 

[1]: https://docs.gitlab.com/omnibus/update/README.html#updating-from-gitlab-6-6-and-higher-to-7-10-or-newer
[2]: https://docs.gitlab.com/ee/raketasks/backup_restore.html#creating-a-backup-of-the-gitlab-system
[3]: https://packages.gitlab.com/gitlab/gitlab-ce
[4]: https://docs.gitlab.com/omnibus/update/README.html#updating-from-gitlab-8-10-and-lower-to-8-11-or-newer
[5]: https://docs.gitlab.com/omnibus/update/README.html#updating-gitlab-10-0-or-newer
