---
title: openstack操作及状态
date: 2024-07-07 10:27:28
description: openstack操作及状态
type: "tags"
comments: true
categories:
- Openstack
- Actions
- Status
tags:
- openstack
---
# Server 

server status 是描述虚拟机当前状态

vm_state 是虚拟机稳定状态

task_state 实例当前发生的情况

虚拟机稳定状态和操作的关系：https://docs.openstack.org/nova/latest/reference/vm-states.html

**InstanceTaskState**
```bash
SCHEDULING = 'scheduling'
BLOCK_DEVICE_MAPPING = 'block_device_mapping'
NETWORKING = 'networking'
SPAWNING = 'spawning'
IMAGE_SNAPSHOT = 'image_snapshot'
IMAGE_SNAPSHOT_PENDING = 'image_snapshot_pending'
IMAGE_PENDING_UPLOAD = 'image_pending_upload'
IMAGE_UPLOADING = 'image_uploading'
IMAGE_BACKUP = 'image_backup'
UPDATING_PASSWORD = 'updating_password'
RESIZE_PREP = 'resize_prep'
RESIZE_MIGRATING = 'resize_migrating'
RESIZE_MIGRATED = 'resize_migrated'
RESIZE_FINISH = 'resize_finish'
RESIZE_REVERTING = 'resize_reverting'
RESIZE_CONFIRMING = 'resize_confirming'
REBOOTING = 'rebooting'
REBOOT_PENDING = 'reboot_pending'
REBOOT_STARTED = 'reboot_started'
REBOOTING_HARD = 'rebooting_hard'
REBOOT_PENDING_HARD = 'reboot_pending_hard'
REBOOT_STARTED_HARD = 'reboot_started_hard'
PAUSING = 'pausing'
UNPAUSING = 'unpausing'
SUSPENDING = 'suspending'
RESUMING = 'resuming'
POWERING_OFF = 'powering-off'
POWERING_ON = 'powering-on'
RESCUING = 'rescuing'
UNRESCUING = 'unrescuing'
REBUILDING = 'rebuilding'
REBUILD_BLOCK_DEVICE_MAPPING = "rebuild_block_device_mapping"
REBUILD_SPAWNING = 'rebuild_spawning'
MIGRATING = "migrating"
DELETING = 'deleting'
SOFT_DELETING = 'soft-deleting'
RESTORING = 'restoring'
SHELVING = 'shelving'
SHELVING_IMAGE_PENDING_UPLOAD = 'shelving_image_pending_upload'
SHELVING_IMAGE_UPLOADING = 'shelving_image_uploading'
SHELVING_OFFLOADING = 'shelving_offloading'
UNSHELVING = 'unshelving'
```
**InstanceState**
```bash
ACTIVE = 'active'
BUILDING = 'building'
PAUSED = 'paused'
SUSPENDED = 'suspended'
STOPPED = 'stopped'
RESCUED = 'rescued'
RESIZED = 'resized'
SOFT_DELETED = 'soft-delete'
DELETED = 'deleted'
ERROR = 'error'
SHELVED = 'shelved'
SHELVED_OFFLOADED = 'shelved_offloaded'
```
**serverStatus**
```bash
_STATE_MAP = {
    vm_states.ACTIVE: {
        'default': 'ACTIVE',
        task_states.REBOOTING: 'REBOOT',
        task_states.REBOOT_PENDING: 'REBOOT',
        task_states.REBOOT_STARTED: 'REBOOT',
        task_states.REBOOTING_HARD: 'HARD_REBOOT',
        task_states.REBOOT_PENDING_HARD: 'HARD_REBOOT',
        task_states.REBOOT_STARTED_HARD: 'HARD_REBOOT',
        task_states.UPDATING_PASSWORD: 'PASSWORD',
        task_states.REBUILDING: 'REBUILD',
        task_states.REBUILD_BLOCK_DEVICE_MAPPING: 'REBUILD',
        task_states.REBUILD_SPAWNING: 'REBUILD',
        task_states.MIGRATING: 'MIGRATING',
        task_states.RESIZE_PREP: 'RESIZE',
        task_states.RESIZE_MIGRATING: 'RESIZE',
        task_states.RESIZE_MIGRATED: 'RESIZE',
        task_states.RESIZE_FINISH: 'RESIZE',
    },
    vm_states.BUILDING: {
        'default': 'BUILD',
    },
    vm_states.STOPPED: {
        'default': 'SHUTOFF',
        task_states.RESIZE_PREP: 'RESIZE',
        task_states.RESIZE_MIGRATING: 'RESIZE',
        task_states.RESIZE_MIGRATED: 'RESIZE',
        task_states.RESIZE_FINISH: 'RESIZE',
        task_states.REBUILDING: 'REBUILD',
        task_states.REBUILD_BLOCK_DEVICE_MAPPING: 'REBUILD',
        task_states.REBUILD_SPAWNING: 'REBUILD',
    },
    vm_states.RESIZED: {
        'default': 'VERIFY_RESIZE',
        # Note(maoy): the OS API spec 1.1 doesn't have CONFIRMING_RESIZE
        # state so we comment that out for future reference only.
        # task_states.RESIZE_CONFIRMING: 'CONFIRMING_RESIZE',
        task_states.RESIZE_REVERTING: 'REVERT_RESIZE',
    },
    vm_states.PAUSED: {
        'default': 'PAUSED',
        task_states.MIGRATING: 'MIGRATING',
    },
    vm_states.SUSPENDED: {
        'default': 'SUSPENDED',
    },
    vm_states.RESCUED: {
        'default': 'RESCUE',
    },
    vm_states.ERROR: {
        'default': 'ERROR',
        task_states.REBUILDING: 'REBUILD',
        task_states.REBUILD_BLOCK_DEVICE_MAPPING: 'REBUILD',
        task_states.REBUILD_SPAWNING: 'REBUILD',
    },
    vm_states.DELETED: {
        'default': 'DELETED',
    },
    vm_states.SOFT_DELETED: {
        'default': 'SOFT_DELETED',
    },
    vm_states.SHELVED: {
        'default': 'SHELVED',
    },
    vm_states.SHELVED_OFFLOADED: {
        'default': 'SHELVED_OFFLOADED',
    },
}

---------
#整理出需要实时监控更新的中间状态
-ACTIVE
-SHUTOFF
-PAUSED
-SUSPENDED
-DELETED
-SOFT_DELETED
*-VERIFY_RESIZE
-SHELVED
-SHELVED_OFFLOADED
*RESCUE
*REBOOT
*HARD_REBOOT
*PASSWORD
*REBUILD
*MIGRATING
*RESIZE
*BUILD
*REVERT_RESIZE
```
**InstancePowerState**
```bash
_UNUSED = '_unused'
NOSTATE = 'pending'
RUNNING = 'running'
PAUSED = 'paused'
SHUTDOWN = 'shutdown'
CRASHED = 'crashed'
SUSPENDED = 'suspended'
```
# image

镜像状态转换：https://docs.openstack.org/glance/train/user/statuses.html

**image status**
```bash
queued
saving
uploading
importing
active
deactivated
killed
deleted
pending_delete
```
tasks status

```bash
pending
processing
success
failure
```
# volume

**Volume statuses**

```bash
# VolumeStatus
*CREATING = 'creating'
AVAILABLE = 'available'
*DELETING = 'deleting'
ERROR = 'error'
*ERROR_DELETING = 'error_deleting'
*ERROR_MANAGING = 'error_managing'
*MANAGING = 'managing'
*ATTACHING = 'attaching'
IN_USE = 'in-use'
*DETACHING = 'detaching'
MAINTENANCE = 'maintenance'
*RESTORING_BACKUP = 'restoring-backup'
*ERROR_RESTORING = 'error_restoring'
RESERVED = 'reserved'
*AWAITING_TRANSFER = 'awaiting-transfer'
*BACKING_UP = 'backing-up'
*ERROR_BACKING_UP = 'error_backing-up'
*ERROR_EXTENDING* = 'error_extending'
*DOWNLOADING = 'downloading'
*UPLOADING = 'uploading'
*RETYPING = 'retyping'
*EXTENDING = 'extending'

# BackupStatus

ERROR = 'error'
ERROR_DELETING = 'error_deleting'
CREATING = 'creating'
AVAILABLE = 'available'
DELETING = 'deleting'
DELETED = 'deleted'
RESTORING = 'restoring'

# SnapshotStatus
ERROR = 'error'
AVAILABLE = 'available'
CREATING = 'creating'
DELETING = 'deleting'
DELETED = 'deleted'
UPDATING = 'updating'
ERROR_DELETING = 'error_deleting'
UNMANAGING = 'unmanaging'
BACKING_UP = 'backing-up'
RESTORING = 'restoring'

# VolumeAttachStatus
ATTACHED = 'attached'
ATTACHING = 'attaching'
DETACHED = 'detached'
RESERVED = 'reserved'
ERROR_ATTACHING = 'error_attaching'
ERROR_DETACHING = 'error_detaching'
DELETED = 'deleted'

# VolumeMigrationStatus
MIGRATING = 'migrating'
ERROR = 'error'
SUCCESS = 'success'
COMPLETING = 'completing'
NONE = 'none'
STARTING = 'starting'

```
# neutron
```bash
NET_STATUS_ACTIVE = 'ACTIVE'
NET_STATUS_BUILD = 'BUILD'
NET_STATUS_DOWN = 'DOWN'
NET_STATUS_ERROR = 'ERROR'

PORT_STATUS_ACTIVE = 'ACTIVE'
PORT_STATUS_BUILD = 'BUILD'
PORT_STATUS_DOWN = 'DOWN'
PORT_STATUS_ERROR = 'ERROR'
PORT_STATUS_NOTAPPLICABLE = 'N/A'

FLOATINGIP_STATUS_ACTIVE = 'ACTIVE'
FLOATINGIP_STATUS_DOWN = 'DOWN'
FLOATINGIP_STATUS_ERROR = 'ERROR'

ROUTER_STATUS_ACTIVE = 'ACTIVE'
ROUTER_STATUS_ALLOCATING = 'ALLOCATING'
ROUTER_STATUS_ERROR = 'ERROR'

```