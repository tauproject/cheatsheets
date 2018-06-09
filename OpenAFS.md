# Tau Hypercomputing Facility
## OpenAFS cheat-sheet

[![Release status: Pre-production][release-status]](#release-status)
![Classification: UNRESTRICTED][classification]
[![CC BY 4.0-licensed][license]](#copyright)

### Contents

* [Release status](#release-status)
* [Server types](#server-types)
	* [Fileserver daemons](#fileserver-daemons)
	* [Database daemons](#database-daemons)
* [Partitions](#partitions)
* [Volume management & replication](#volume-management--replication)
	* [List volumes and replication sites](#list-volumes-and-replication-sites)
	* [Scan for volumes on a fileserver](#scan-for-volumes-on-a-fileserver)
	* [Create a new volume](#create-a-new-volume)
	* [Display a volume’s quota](#display-a-volumes-quota)
	* [Modify a volume’s quota](#modify-a-volumes-quota)
	* [Add a replication site to an existing volume](#add-a-replication-site-to-an-existing-volume)
	* [Remove a replication site from a volume](#remove-a-replication-site-from-a-volume)
	* [Move a volume to a different server or partition](#move-a-volume-to-a-different-server-or-partition)
	* [Convert a read-only replica of a volume to read-write](#convert-a-read-only-replica-of-a-volume-to-read-write)
	* [Force a volume to be online or offline](#force-a-volume-to-be-online-or-offline)
	* [Delete a volume](#delete-a-volume)
* [Mount-points](#mount-points)
	* [Mount a volume within the AFS filespace](#mount-a-volume-within-the-afs-filespace)
	* [Mount a foreign cell within the AFS filespace](#mount-a-foreign-cell-within-the-afs-filespace)
	* [Remove a mount-point](#remove-a-mount-point)
	* [Inspect a mount-point](#inspect-a-mount-point)
* [User management](#user-management)
	* [Pre-defined AFS groups](#pre-defined-afs-groups)
	* [List AFS users](#list-afs-users)
	* [List AFS groups](#list-afs-groups)
	* [List members of an AFS group](#list-members-of-an-afs-group)
	* [Create an AFS user (assign a user ID to a principal)](#create-an-afs-user-assign-a-user-id-to-a-principal)
	* [Create an AFS group](#create-an-afs-group)
	* [Add an AFS user to a group](#add-an-afs-user-to-a-group)
	* [Remove an AFS user from a group](#remove-an-afs-user-from-a-group)
	* [Delete an AFS user](#delete-an-afs-user)
	* [Delete an AFS group](#delete-an-afs-group)
* [Access control lists (ACLs)](#access-control-lists-acls)
	* [Display the ACL for a path](#display-the-acl-for-a-path)
	* [Add a new access control entry to an ACL](#add-a-new-access-control-entry-to-an-acl)
* [Copyright](#copyright) 

### Release status

| Release track           | Tag/branch    |
|-------------------------|---------------|
| Long-term support (LTS) | Not available |
| Stable                  | Not available |
| Pre-production          | [`shimmer`](https://github.com/tauproject/cheatsheets/tree/shimmer/master) |
| Legacy                  | Not available |

### Server types

AFS servers are divided into two categories, _fileserver_ and _database server_, although a host may be both a file- and database server if it runs both sets of daemons.

On all server types, the “Basic Overseer Server” ([`bosserver`](http://docs.openafs.org/Reference/8/bosserver.html)) is used to orchestrate running daemons, and is managed using the [`bos`](http://docs.openafs.org/Reference/8/bos.html) utility.

#### Fileserver daemons

* File server: [`fileserver`](http://docs.openafs.org/Reference/8/fileserver.html) or [`dafileserver`](http://docs.openafs.org/Reference/8/dafileserver.html)
* Volume server: [`volserver`](http://docs.openafs.org/Reference/8/volserver.html) or [`davolserver`](http://docs.openafs.org/Reference/8/davolserver.html)
* File server: [`salvageserver`](http://docs.openafs.org/Reference/8/salvageserver.html) or [`dasalvager`](http://docs.openafs.org/Reference/8/dasalvager.html)

#### Database daemons

* Volume location server [`vlserver`](http://docs.openafs.org/Reference/8/vlserver.html)
* Protection database server [`ptserver`](http://docs.openafs.org/Reference/8/ptserver.html)

### Partitions

AFS is a type of overlay filesystem: rather than utilising block devices directly, it stores volume contents in partitions on fileservers.

When the File Server ([`fileserver`](http://docs.openafs.org/Reference/8/fileserver.html)) starts up, it scans the root directory for subdirectories named `/vicepX...`, where `X...` is the identifier of the partition (the first AFS partition is `a`, the second is `b`, the twenty-seventh is `aa`, and so on).

Each `/vicepX...` directory should be an OS-native mount-point, and the filesystem mounted there should not be used for other purposes. A non-mount-point directory can be adopted as an AFS partition, provided it conforms to the naming scheme, by creating a file named `AlwaysAttach` within it.

A partition can be forcibly ignored by creating a file named `NeverAttach` within it.

If both `AlwaysAttach` and `NeverAttach` are present, `AlwaysAttach` takes precedence.

### Volume management & replication

Volume management is accomplished via the [`vos`](http://docs.openafs.org/Reference/1/vos.html) utility, which manages the Volume Server ([`volserver`](http://docs.openafs.org/Reference/8/volserver.html)), part of the _filesystem server suite_, and [`vlserver`](http://docs.openafs.org/Reference/8/vlserver.html), which maintains the Volume Location Database (VLDB).

When invoked as `root` on an AFS fileserver, `vos` commands accept the `-localauth` flag to use the host’s AFS key instead of the AFS token cache.

Volumes are stored on AFS partitions, which must be mounted on `/vicepX` on a fileserver, where `X` is the letter representing the partition ID (the first AFS partition is `a`, the second is `b`, the twenty-seventh is `aa`, and so on).

#### List volumes and replication sites

Use [`vos listvldb`](http://docs.openafs.org/Reference/1/vos_listvldb.html) to list the contents of the VLDB.

```sh
$ vos listvldb [VOLUME] [-server SERVER]
```

If `VOLUME` or `SERVER` is specified, the listing shall be restricted to specified volume and/or fileserver.

#### Scan for volumes on a fileserver

Use [`vos listvol`](http://docs.openafs.org/Reference/1/vos_listvol.html) to scan for volumes on a server, bypassing the VLDB.

```sh
$ vos listvol SERVER [-partition PARTITION] [-fast|-extended] [-long] [-format]
```

If `PARTITION` is specified, only that partition will be scanned.

The `-fast` and `-extended` options are mutually exclusive and control the type of volume scan performed.

The `-long` option results in detailed information about the volumes located.

The `-format` option produces output in a consistent format that can be parsed by other utilities.

#### Create a new volume

Use [`vos create`](http://docs.openafs.org/Reference/1/vos_create.html) to create a new volume.

```sh
$ vos create SERVER PARTITION NAME [-maxquota SIZE]
```

`SIZE`, if specified, is in kilobyte units. If unspecified, the default quota is 5000 KB. To create a volume with no quota, specify `-maxquota 0`.

##### Example:

```sh
# Create a new volume 'myvol' on partition 'd' of fileserver 'afs01'

$ vos create afs01 d myvol
Volume 536870945 created on partition /vicepd of afs01
```

#### Display a volume’s quota

Use [`fs listquota`](http://docs.openafs.org/Reference/1/fs_listquota.html) to display quota information.

**Note:** the volume must be mounted to display the quota

```sh
$ fs listquota [DIR]
```

If `DIR` is not specified, it defaults to the current working directory.

#### Modify a volume’s quota

Use [`fs setquota`](http://docs.openafs.org/Reference/1/fs_setquota.html) to modify quota information.

**Note:** the volume must be mounted to modify the quota

```sh
$ fs setquota [DIR] -max [SIZE]
```

As with `vos create`, `SIZE` is specified in kilobyte units.

#### Add a replication site to an existing volume

Use [`vos addsite`](http://docs.openafs.org/Reference/1/vos_addsite.html) to add replication sites.

A volume has one read/write site, but can have multiple read-only replication sites. Note that replication is not useful for volumes which change frequently (such as home directories).

```sh
$ vos addsite SERVER PARTITION VOLUME
```

##### Example:

```sh
# Add partition 'b' on fileserver 'afs02' as a read-only replication site
# of volume 'myvol'

$ vos addsite afs02 b myvol
```

#### Remove a replication site from a volume

Use [`vos remove`](http://docs.openafs.org/Reference/1/vos_remove.html) to remove replication sites.

```sh
$ vos remove SERVER PARTITION VOLUME
```

##### Example:

```sh
# Remove the replication site on partition 'b' on fileserver 'afs02' for
# the volume 'myvol'

$ vos remove afs02 b myvol
```

#### Move a volume to a different server or partition

Use [`vos move`](http://docs.openafs.org/Reference/1/vos_move.html) to move volume replicas between servers or partitions.

```sh
$ vos move -id VOLUME -froms[erver] SERVER -fromp[artition] PARTITION -tos[server] NEW-SERVER -top[artition] NEW-PARTITION [-live]
```

Note that `SERVER` and `NEW-SERVER` must both be specified even if the volume is not being moved between servers.

The `-live` option locks the volume and performs live-migration without creating a temporary copy; this may be required if the source partition is low on space. It should not generally be used otherwise.


#### Convert a read-only replica of a volume to read-write

Use [`vos convertROtoRW`](http://docs.openafs.org/Reference/1/vos_convertROtoRW.html) to convert a read-only replica of a volume to the primary (read-write) site.

**Note:** the `convertROtoRW` sub-command is case-sensitive and must be entered as shown here.

```sh
$ vos convertROtoRW SERVER PARTITION VOLUME
```

#### Force a volume to be online or offline

Use [`vos online`](http://docs.openafs.org/Reference/1/vos_online.html) and [`vos offline`](http://docs.openafs.org/Reference/1/vos_offline.html) to force a volume online or offline.

```sh
$ vos online SERVER PARTITION VOLUME

$ vos offline SERVER PARTITION VOLUME [-busy -sleep SECONDS]
```

When taking a volume offline temporarily, the `-busy` and `-sleep` options may be more appropriate.

#### Delete a volume 

Use [`vos remove`](http://docs.openafs.org/Reference/1/vos_remove.html) to delete volumes.
 
```sh
$ vos remove -id VOLUME
```

Note that a volume isn’t fully deleted until all of its replicas are removed. `vos remove -id VOLUME` will remove the read/write primary (and the backup volume, if it exists), but will not remove the read-only replicas.

### Mount-points

#### Mount a volume within the AFS filespace

Use [`fs mkmount`](http://docs.openafs.org/Reference/1/fs_mkmount.html) to create a mount-point for a volume.

```sh
$ fs mkmount PATH VOLUME
```

#### Mount a foreign cell within the AFS filespace

Use [`fs mkmount`](http://docs.openafs.org/Reference/1/fs_mkmount.html) to create a mount-point for a cell.

```sh
$ fs mkmount PATH -cell CELL
```

#### Remove a mount-point

Use [`fs rmmount`](http://docs.openafs.org/Reference/1/fs_rmmount) to remove a mount-point for a volume.

```
$ fs rmmount PATH
```

#### Inspect a mount-point

Use [`fs lsmount`](http://docs.openafs.org/Reference/1/fs_lsmount) to inspect a mount-point.

```
$ fs lsmount PATH
```

### User management

AFS uses **Kerberos** to perform authentication, and operates a _Protection Database_ for authorisation/access control.

The [`pts`](http://docs.openafs.org/Reference/1/pts.html) utility manages the Protection Server ([`ptserver`](http://docs.openafs.org/Reference/8/ptserver.html)), which maintains Kerberos to AFS user mappings and group information in the Protection Database.

**Note:** In all cases where a principal is required, it must be specified without the realm suffix (i.e., `@REALM.NAME`). If the principal includes an instance, it should be specified in `NAME.INSTANCE` form rather than `NAME/INSTANCE` as is usual in Kerberos V.

When invoked as `root` on an AFS fileserver, `pts` commands accept the `-localauth` flag to use the host’s AFS key instead of the AFS token cache.

AFS users and groups _share a unified naming and ID space_: users always have positive integer IDs, groups always have negative integer IDs.

#### Pre-defined AFS groups

The following pre-defined groups exist in every AFS cell:

| Group                   | Description                                                 |
|-------------------------|-------------------------------------------------------------|
| `system:administrators` | AFS administrators                                          |
| `system:backup`         | AFS backup system                                           |
| `system:anyuser`        | Any AFS user, including anonymous clients and foreign cells |
| `system:authuser`       | Any user authenticated in this cell                         |
| `system:ptsviewers`     | Users granted permission to view the protection database    |

#### List AFS users

Use [`pts listentries`](http://docs.openafs.org/Reference/1/pts_listentries.html) to list AFS users.

```sh
$ pts listentries [-users]
```

#### List AFS groups

Use [`pts listentries`](http://docs.openafs.org/Reference/1/pts_listentries.html) to list AFS groups.

```sh
$ pts listentries -groups
```

#### List members of an AFS group

Use [`pts membership`](http://docs.openafs.org/Reference/1/pts_membership.html) to list AFS group members.

```sh
$ pts membership [OWNER:]GROUP
```

#### List AFS groups owned by another user or group

Use [`pts listowned`](http://docs.openafs.org/Reference/1/pts_listowned.html) to list AFS groups owned by a user or group.

```sh
$ pts listowned PRINCIPAL
```

#### Create an AFS user (assign a user ID to a principal)

Use [`pts createuser`](http://docs.openafs.org/Reference/1/pts_createuser.html) to create an entry in the protection database for a user, mapping a numeric user ID to a principal.

```sh
$ pts createuser PRINCIPAL UID
```

##### Example:

```sh
$ pts createuser admin.admin 1
```

#### Create an AFS group

Use [`pts creategroup`](http://docs.openafs.org/Reference/1/pts_creategroup.html) to create a new AFS group.

```sh
$ pts creategroup -name [OWNER:]GROUP [-owner OWNER] [-id -GID]
```

##### Notes:

1. If invoked by a member of `system:administrators`, then omitting the `OWNER` will create a “prefix-less group” instead of implicitly prefixing the group name with the current user’s name.
2. Only members of `system:administrators` can create prefix-less groups.
3. If `OWNER:` is specified as part of the group name, the `-owner OWNER` argument is redundant (and if specified, name the same `OWNER`).
4. Group IDs (`GID`) are always specified as negative numbers.

#### Add an AFS user to a group

Use [`pts adduser`](http://docs.openafs.org/Reference/1/pts_adduser.html) to add an AFS user to a group.

```sh
$ pts adduser PRINCIPAL GROUP
```

#### Remove an AFS user from a group

Use [`pts removeuser`](http://docs.openafs.org/Reference/1/pts_removeuser.html) to remove an AFS user from a group.

```sh
$ pts removeuser PRINCIPAL GROUP
```

#### Delete an AFS user

Use [`pts delete`](http://docs.openafs.org/Reference/1/pts_delete.html) to delete an AFS user.

```sh
$ pts delete PRINCIPAL
```

#### Delete an AFS group

Use [`pts delete`](http://docs.openafs.org/Reference/1/pts_delete.html) to delete an AFS group.

```sh
$ pts delete [OWNER:]GROUP
```

### Access control lists (ACLs)

**Note:** In OpenAFS, ACLs are set on _directories_, not individual files.

AFS ACLs are managed through the [`fs`](http://docs.openafs.org/Reference/1/fs.html) utility.

##### Directory permissions

The following permissions apply to the directory the ACL is assigned to:

| Character | Permission                               |
|-----------|------------------------------------------|
| `l`       | Lookup (list contents and read ACLs)     |
| `i`       | Insert (create files/subdirectories)     |
| `d`       | Delete files/subdirectories              |
| `a`       | Administer (modify this directory’s ACL) |

##### File permissions

The following permissions apply to files _within_ the directory:

| Character | Permission                               |
|-----------|------------------------------------------|
| `r`       | Read contents of files/subdirectories    |
| `w`       | Write (modify files/subdirectories)      |
| `k`       | Lock files within the directory          |

##### Permission aliases

Additionally, the following symbolic aliases are recognised when setting ACLs (via `fs setacl`):

| Alias   | Expansion | Meaning                                 |
|---------|-----------|-----------------------------------------|
| `none`  | _(empty)_ | Explicitly grant no permissions         |
| `read`  | `rl`      | Read directory and its contents         |
| `write` | `rlidwk`  | All permissions except `a` (administer) |
| `all`   | `rlidwka` | All permissions                         |


#### Display the ACL for a path

Use [`fs listacl`](http://docs.openafs.org/Reference/1/fs_listacl.html) to display the access control list (ACL) for a path.

```sh
$ fs listacl [PATH]
```

If unspecified, `PATH` defaults to the current working directory.

#### Add a new access control entry to an ACL

Use [`fs setacl`](http://docs.openafs.org/Reference/1/fs_setacl.html) to modify a directory’s access control list.

```sh
$ fs setacl PATH [USER|GROUP PERMISSIONS ...] [-clear]
```

If `-clear` is specified, the newly-specified ACL will **replace** any existing ACLs.

`PERMISSIONS` is specified as a sequence of lower-case characters or a symbolic alias indicating the permissions that should be granted, corresponding to the tables provided above.

##### Examples:

```sh
# Allow any user authenticated in the current cell to list and read
# the contents of this directory

$ fs setacl . system:authuser read

# Grant all permissions to administrator, write permission to a user
# or group named 'services', and remove any existing ACLs on a
# directory named 'dns'

$ fs setacl dns system:administrators all services write -clear

# Allow any user, including anonymous users and members of foreign
# cells to, access the directory 'public'

$ fs setacl public system:anyuser read
```

### Copyright

Copyright © 2018.

Licensed under the [Creative Commons Attribution 4.0 International (CC BY 4.0) license](https://creativecommons.org/licenses/by/4.0/) (the “License”); you may not use this file except in compliance with the License.

You may obtain a copy of the License at:

https://creativecommons.org/licenses/by/4.0/

Unless required by applicable law or agreed in writing, files distributed
under the License are done so on an **“as-is” basis, without warranties of
any kind**, either express or implied. See the License for the specific
language governing permissions and limitations under the License.

```
@(#) $Tau: cheatsheets/OpenAFS.md $
```

[license]: https://img.shields.io/badge/license-CC%20BY%204.0-blue.svg?style=flat-square
[release-status]: https://img.shields.io/badge/release%20status-Pre--production-yellow.svg?style=flat-square
[classification]: https://img.shields.io/badge/classification-UNRESTRICTED-brightgreen.svg?style=flat-square
