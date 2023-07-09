



```
This is your host IP address: 172.25.70.158
This is your host IPv6 address: ::1
Horizon is now available at http://172.25.70.158/dashboard
Keystone is serving at http://172.25.70.158/identity/
The default users are: admin and demo
The password: nomoresecret

Services are running under systemd unit files.
For more information see: 
https://docs.openstack.org/devstack/latest/systemd.html

DevStack Version: 2023.2
Change: 4dfb67a831686279acd66f65e51beba42f675c91 Merge "Update DEVSTACK_SERIES to 2023.2" 2023-03-17 01:59:49 +0000
OS Version: Ubuntu 22.04 jammy



root Huawei12#$%
公网ip
120.26.10.26
```









```
Traceback (most recent call last): File "/opt/stack/nova/nova/compute/manager.py", line 2175, in _prep_block_device driver_block_device.attach_block_devices( 
File "/opt/stack/nova/nova/virt/block_device.py", line 936, in attach_block_devices _log_and_attach(device) 
File "/opt/stack/nova/nova/virt/block_device.py", line 933, in _log_and_attach bdm.attach(*attach_args, **attach_kwargs) 
File "/opt/stack/nova/nova/virt/block_device.py", line 831, in attach self.volume_id, self.attachment_id = self._create_volume( File "/opt/stack/nova/nova/virt/block_device.py", line 435, in _create_volume self._call_wait_func(context, wait_func, volume_api, vol['id']) 
File "/opt/stack/nova/nova/virt/block_device.py", line 785, in _call_wait_func with excutils.save_and_reraise_exception(): 
File "/usr/local/lib/python3.10/dist-packages/oslo_utils/excutils.py", line 227, in __exit__ self.force_reraise() File "/usr/local/lib/python3.10/dist-packages/oslo_utils/excutils.py", line 200, in force_reraise raise self.value File "/opt/stack/nova/nova/virt/block_device.py", line 783, in _call_wait_func wait_func(context, volume_id) File "/opt/stack/nova/nova/compute/manager.py", line 1792, in _await_block_device_map_created raise exception.VolumeNotCreated(volume_id=vol_id, nova.exception.VolumeNotCreated: Volume 348c787a-f70f-49aa-8603-983c340a3bbf did not finish being created even after we waited 3 seconds or 2 attempts. And its status is error. During handling of the above exception, another exception occurred: Traceback (most recent call last): File "/opt/stack/nova/nova/compute/manager.py", line 2825, in _build_resources block_device_info = self._prep_block_device(context, instance, File "/opt/stack/nova/nova/compute/manager.py", line 2194, in _prep_block_device raise exception.InvalidBDM(str(ex)) nova.exception.InvalidBDM: Volume 348c787a-f70f-49aa-8603-983c340a3bbf did not finish being created even after we waited 3 seconds or 2 attempts. And its status is error. During handling of the above exception, another exception occurred: Traceback (most recent call last): File "/opt/stack/nova/nova/compute/manager.py", line 2428, in _do_build_and_run_instance self._build_and_run_instance(context, instance, image, File "/opt/stack/nova/nova/compute/manager.py", line 2635, in _build_and_run_instance with excutils.save_and_reraise_exception(): File "/usr/local/lib/python3.10/dist-packages/oslo_utils/excutils.py", line 227, in __exit__ self.force_reraise() File "/usr/local/lib/python3.10/dist-packages/oslo_utils/excutils.py", line 200, in force_reraise raise self.value File "/opt/stack/nova/nova/compute/manager.py", line 2590, in _build_and_run_instance with self._build_resources(context, instance, File "/usr/lib/python3.10/contextlib.py", line 135, in __enter__ return next(self.gen) File "/opt/stack/nova/nova/compute/manager.py", line 2837, in _build_resources raise exception.BuildAbortException(instance_uuid=instance.uuid, nova.exception.BuildAbortException: Build of instance 21aaada2-9383-4d05-8e21-fd01f82f28b2 aborted: Volume 348c787a-f70f-49aa-8603-983c340a3bbf did not finish being created even after we waited 3 seconds or 2 attempts. And its status is error. 
```

