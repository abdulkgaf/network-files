# network-files
This lab demonstrates network file sharing configuration and management in a Windows Server Active Directory environment, including creating shared folders with various permissions, testing access controls, and managing security groups for file access.

# Network File and Sharing Lab Documentation

## Lab Overview
This lab demonstrates network file sharing configuration and management in a Windows Server Active Directory environment, including creating shared folders with various permissions, testing access controls, and managing security groups for file access.

## Prerequisites
- Azure subscription with DC-1 and Client-1 VMs configured
- DC-1: Windows Server with Active Directory Domain Services
- Client-1: Windows client joined to the domain
- Domain admin account (mydomain.com\jane_admin)
- Normal domain user account (mydomain\<someuser>)

## Environment Setup
1. **Start Azure VMs**: Ensure DC-1 and Client-1 VMs are powered on in Azure Portal
2. **Verify Network Connectivity**: Confirm both VMs can communicate on the virtual network

---

## Create some sample file shares with various permissions

### Steps

#### 1. Connect/log into DC-1 as your domain admin account (mydomain.com\jane_admin)
- RDP to DC-1 using domain admin credentials
- Username: `mydomain.com\jane_admin`
- Password: [Your configured password]

#### 2. Connect/log into Client-1 as a normal user (mydomain\<someuser>)
- RDP to Client-1 using normal user credentials  
- Username: `mydomain\<someuser>`
- Password: [Your configured password]

#### 3. On DC-1, on the C:\ drive, create 4 folders: "read-access", "write-access", "no-access", "accounting"

![Screenshot 2025-05-28 150935](https://github.com/user-attachments/assets/15424357-36d8-4334-a11c-4d06d052b6a6)

On DC-1:
1. Open **File Explorer**
2. Navigate to **C:\** drive
3. Create the following folders:
   - Right-click → **New** → **Folder** → Name: `read-access`
   - Right-click → **New** → **Folder** → Name: `write-access`
   - Right-click → **New** → **Folder** → Name: `no-access`
   - Right-click → **New** → **Folder** → Name: `accounting`

#### 4. Set the following permissions (share the folder)

![Screenshot 2025-05-28 151023](https://github.com/user-attachments/assets/d4a258c3-bf30-434b-aceb-4350e9061ed6)

##### Folder: "read-access", Group: "Domain Users", Permission: "Read"
1. Right-click **read-access** folder → **Properties**
2. Go to **Sharing** tab → **Advanced Sharing**
3. Check **Share this folder**
4. Click **Permissions**
5. Remove **Everyone** group
6. Add **Domain Users** group with **Read** permission
7. Click **OK** → **OK** → **Close**

##### Folder: "write-access", Group: "Domain Users", Permissions: "Read/Write"
1. Right-click **write-access** folder → **Properties**
2. Go to **Sharing** tab → **Advanced Sharing**
3. Check **Share this folder**
4. Click **Permissions**
5. Remove **Everyone** group
6. Add **Domain Users** group with **Full Control** permission
7. Click **OK** → **OK** → **Close**

##### Folder: "no-access", Group: "Domain Admins", Permissions: "Read/Write"
1. Right-click **no-access** folder → **Properties**
2. Go to **Sharing** tab → **Advanced Sharing**
3. Check **Share this folder**
4. Click **Permissions**
5. Remove **Everyone** group
6. Add **Domain Admins** group with **Full Control** permission
7. Click **OK** → **OK** → **Close**

##### (Skip accounting for now)

---
![Screenshot 2025-05-28 151101](https://github.com/user-attachments/assets/ba7a563f-fc3b-4bb4-8d95-47f6561e7259)

## Attempt to access file shares as a normal user

### Steps

#### 1. On Client-1, navigate to the shared folder (start, run, \\dc-1)
On Client-1:
1. Press **Windows Key + R**
2. Type: `\\dc-1`
3. Press **Enter**

#### 2. Try to access the folders you just created. Which folders can you access? Which folders can you create stuff in? Does it make sense?

**Test read-access folder:**
- Double-click **read-access** folder
- **Observation**: Can access and view contents, but cannot create files

**Test write-access folder:**
- Double-click **write-access** folder  
- Try to create a new file or folder
- **Observation**: Can access, view contents, and create new files/folders

**Test no-access folder:**
- Double-click **no-access** folder
- **Observation**: Access denied - cannot access folder at all

---

## Create an "ACCOUNTANTS" Security Group, assign permissions, and test access

### Steps

#### 1. Go back to DC-1, in Active Directory, create a security group called "ACCOUNTANTS"
On DC-1:
1. Open **Active Directory Users and Computers**
2. Expand domain → Right-click **Users** container
3. Select **New** → **Group**
4. Configure group:
   - **Group name**: `ACCOUNTANTS`
   - **Group scope**: **Global**
   - **Group type**: **Security**
5. Click **OK**

![Screenshot 2025-05-28 151144](https://github.com/user-attachments/assets/577d91f0-6967-4228-8de7-217fff5e7513)

#### 2. On the "accounting" folder you created earlier, set the following permissions:

##### Folder: "accounting", Group: "ACCOUNTANTS", Permissions: "Read/Write"
1. Right-click **accounting** folder → **Properties**
2. Go to **Sharing** tab → **Advanced Sharing**
3. Check **Share this folder**
4. Click **Permissions**
5. Remove **Everyone** group
6. Add **ACCOUNTANTS** group with **Full Control** permission
7. Click **OK** → **OK** → **Close**

#### 3. On Client-1, as <someuser>, try to access the accountants folder. It should fail.
On Client-1:
1. Navigate to `\\dc-1`
2. Double-click **accounting** folder
3. **Observation**: Access denied - user is not member of ACCOUNTANTS group

#### 4. Log out of Client-1 as <someuser>
On Client-1:
1. Click **Start** → **User Account** → **Sign out**

#### 5. On DC-1, make <someuser> a member of the "ACCOUNTANTS" Security Group
On DC-1:
1. In **Active Directory Users and Computers**
2. Navigate to **Users** container
3. Double-click **ACCOUNTANTS** group
4. Go to **Members** tab
5. Click **Add**
6. Type: `<someuser>`
7. Click **Check Names** → **OK** → **OK**

#### 6. Sign back into Client-1 as <someuser> and try to access the "accounting" share in \\DC-1\ - Does it work now?
On Client-1:
1. Log back in as `mydomain\<someuser>`
2. Navigate to `\\dc-1`
3. Double-click **accounting** folder
4. **Observation**: Can now access the folder and create files - group membership works!

---

## Key Learning Points

### Network File Sharing Concepts
- **Share Permissions**: Control access at the network level
- **Security Groups**: Used to organize users and assign permissions efficiently
- **UNC Paths**: Universal Naming Convention for network resources (\\server\share)
- **Permission Inheritance**: How permissions flow from groups to users

### Permission Types
- **Read**: View files and folder contents only
- **Read/Write (Full Control)**: Complete access including create, modify, delete
- **No Access**: Explicitly denied access to resources

### Security Group Management
- **Global Groups**: Can contain users from the same domain
- **Security vs Distribution**: Security groups used for permissions, Distribution for email
- **Group Membership**: Users inherit permissions from all groups they belong to

### Best Practices
- Use security groups instead of individual user permissions
- Apply principle of least privilege
- Test permissions with different user accounts
- Document folder purposes and access requirements

## Troubleshooting Tips

### Common Issues
1. **Access Denied Errors**
   - Check user is member of appropriate security group
   - Verify share permissions are correctly configured
   - Confirm user has logged off/on after group membership changes

2. **Cannot Find Network Path**
   - Verify server name resolution (\\dc-1)
   - Check network connectivity between client and server
   - Ensure file sharing services are running

3. **Group Membership Not Taking Effect**
   - User must log off and back on for new group membership
   - Check group membership in Active Directory
   - Verify group has correct permissions on shared folder

### Verification Commands
```cmd
# View available shares on server
net view \\dc-1

# Check current user's group memberships
whoami /groups

# Map network drive
net use Z: \\dc-1\share-name

# Test network connectivity
ping dc-1
```

## Lab Cleanup
- **Do NOT delete the Azure VMs** - they will be used for future labs
- To save costs when not in use: Stop the VMs in Azure Portal
- Network shares and security groups can remain configured for future exercises

## Next Steps
The file sharing infrastructure and security groups created in this lab will be used in upcoming Group Policy and advanced security labs. Understanding network file access and group-based permissions is fundamental for enterprise Windows administration.
