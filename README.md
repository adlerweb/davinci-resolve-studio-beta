Beware: This is only a rough patch for newer versions. It only works for studio. You may need to apply manual patches to make it start (see issues).

# Permissions

To load software licenses or optimize neural engins, the user must have write permissions to the resolve program folder. Ideally we would create a group to manage this properly, but in the meantime something like `chgrp -R youruser /opt/resolve && chmod -R g+rwx /opt/resolve` should work.
