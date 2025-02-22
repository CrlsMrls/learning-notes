# AppArmor in Kubernetes  

AppArmor (Application Armor) provides Mandatory Access Control (MAC) by defining policies that limit how processes can interact with the system. It is more fine-grained than [Seccomp](08-seccomp.md) because it controls file access, capabilities, and networking.

AppArmor helps isolate containers by preventing them from interacting with sensitive parts of the system. Use cases:
- Restrict files and directories a container can access.
- What system calls a container can make (e.g., preventing it from opening raw sockets).
- Preventing certain types of network access.


## 📌 Understanding AppArmor in Kubernetes  

Kubernetes allows to assign AppArmor profiles to Pods to control the capabilities of containers. These profiles define rules that restrict what containers can do, preventing them from performing actions that could compromise the node or cluster.

### 🔐 Key Concepts  
- **AppArmor Profile**: Defines a set of rules that restrict the actions a container can perform. It can allow or deny system calls, access to files, and more.  
- **AppArmor Namespaces**: The security context of a container or pod determines whether AppArmor profiles are applied. Profiles are enforced per container based on the configured security context.  
- **Profile Modes**:  
  - **Enforce mode**: The profile is actively restricting the container based on the rules.  
  - **Audit mode**: The profile logs all actions but does not actively block any operations.  


## Managing AppArmor Profiles  

You can manage and create AppArmor profiles using `apparmor-utils`. Here’s how you can create and load an AppArmor profile for use with Kubernetes:

1. **Create a custom AppArmor profile**:  
   First, write a profile that specifies allowed actions and access restrictions. For example:

   ```bash
   profile my-container-profile flags=(attach_disconnected) {
       # Allow read-only access to the container's filesystem
       /etc/** r,
       /usr/** r,

        # Allow the container to write to /tmp
        /tmp/** rw,

        # Deny access to the host's filesystem
        deny /proc/sys/** rw,
        deny /sys/** rw,
        deny /proc/sysrq-trigger rw,
        deny /proc/irq/** rw,
        deny /proc/bus/** rw,
        deny /proc/fs/** rw,
        ...
   }
   ```

2. **Load the profile**:  
   Use the following command to load the profile into the AppArmor system:

   ```bash
   sudo apparmor_parser -r /etc/apparmor.d/my-container-profile
   ```

   The name of the profile and the file name should match the name of the profile specified in the `appArmorProfile` field.

3. **Apply the profile to the container**:  
   Reference the profile in the pod or container manifest as shown above.

## 🛠️ Enabling AppArmor in Kubernetes  

To use AppArmor in Kubernetes, enable the AppArmor kernel module (kmod) or ensure that AppArmor uses `eBPF` mode.
Kubernetes uses the `securityContext` field to define the AppArmor profile for containers.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
spec:
  containers:
  - name: my-container
    image: my-image
    securityContext:
      appArmorProfile:
        type: Localhost
        localhostProfile: my-custom-profile
```

In this example, the container **`my-container`** will use the custom AppArmor profile **`my-custom-profile`**.


## 📚 References  

- [AppArmor Official Documentation](https://wiki.ubuntu.com/AppArmor)  
- [Kubernetes Security Context Documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)  
- [AppArmor Profiles in Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/apparmor/)  
