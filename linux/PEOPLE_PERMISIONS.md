## 👥 People and Permissions

Notes on managing users, groups, and access levels in the Linux terminal.

### 1. User & Group Management
* `useradd`: Creates a new user account.
  * *Example:* `sudo useradd -m username` (the `-m` creates a home directory).
* `usermod`: Modifies an existing user (e.g., changing their shell or adding them to a group).
  * *Example:* `sudo usermod -aG sudo username` (adds user to the sudoers group).
* `passwd`: Used to set or change a user's password.
* `/etc/skel/`: A directory containing "skeleton" files (like `.bashrc`) that are automatically copied to a new user's home directory when their account is created.
* `group`: Groups allow you to manage permissions for multiple users at once. Managed via `/etc/group`.

### 2. Permissions & Ownership
* `chmod` (Change Mode): Changes file/directory permissions.
  * *Symbolic:* `chmod g+w,o-r file` (Add group write, remove other read).
  * *Numeric:* `chmod 755 file` (rwxr-xr-x).
* `chown` (Change Owner): Changes the owner and/or group of a file.
  * *Example:* `sudo chown user:group filename`.
* `umask` (User Mask): Defines the default permissions for *newly created* files and folders.
  * *Example:* A umask of `022` typically results in `644` for new files.

### 3. Privilege & Security
* `sudoers`: The configuration file (found at `/etc/sudoers`) that defines which users have `sudo` privileges. 
  * *Tip:* Always use `sudo visudo` to edit this file to prevent syntax errors that could lock you out.
* `user`: Generally refers to the individual accounts. You can check your current identity with `whoami` or `id`.

### 4. System Monitoring (Who is doing what?)
* `w`: Shows who is currently logged into the system and what they are doing (their current process).
* `last`: Shows a history of the last logged-in users and their login/logout times.
