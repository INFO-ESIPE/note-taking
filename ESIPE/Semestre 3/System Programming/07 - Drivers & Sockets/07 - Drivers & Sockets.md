[[System Programming]]
****
**Table of Contents**
```table-of-contents
```

****
## From syscall to driver

```c
/*
* Read up to “count” bytes from “file” starting from “pos” into “buf”.
* Return error or number of bytes read
*/
ssize_t vfs_read(struct file *file, char __user *buf, size_t count, loff_t *pos){
	ssize_t ret;
	// Check if we are allowed to read this file
	if (!(file->f_mode & FMODE_READ)) {
		return -EBADF;
	}
	
	// Check if file has read methods
	if (!file->f_op || (!file->f_op->read && !file->f_op->aio_read)) {
		return -EINVAL;
	}

	// Check whether we can write to buf
	if (unlikely(!access_ok(VERIFY_WRITE, buf, count))) {
		return -EFAULT;
	}

	ret = rw_verify_area(READ, file, pos, count);
	if (ret >= 0) { // Check whether we read from a valid range in the file.
		count = ret;

		// We use the driver's provided read function if there is one,
		// otherwise we use use do_sync_read()
		if (file->f_op->read) {
			ret = file->f_op->read(file, buf, count, pos);
		} else {
			ret = do_sync_read(file, buf, count, pos);
		}

		// Notify parent of this file that the file was read
		if (ret > 0) {
			fsnotify_access(file->f_path.dentry);

			// Update the number of bytes read by “current” task
			// (for scheduling purposes)
			add_rchar(current, ret);
		}

		// Update the number of read syscalls by “current” task 
		// (for scheduling purposes)
		inc_syscr(current);
	}
	return ret;
}
```
> The `file` parameter corresponds to the **Open File Description**, which is a structure containing all information about a specific file. Most importantly, it indicates the inode and `loff_t` (the current position within the file). 


***
## Driver

