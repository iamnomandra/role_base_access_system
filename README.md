## 🚀 RBAS (Role-Based Access System) With Roles And CRUD (Create, Read, Update & Delete) Permissions

A robust Role-Based Access System (RBAS) built using **.NET Core**, **Razor Pages**, and **SQL**, designed to provide dynamic UI rendering and fine-grained access control.

---

## ✨ Features

- 🔹 **Database-Driven Dynamic Menu Rendering**
  - Menu and submenu are generated dynamically from database
  - Role-based visibility control

- 🔹 **Single Page Access Control**
  - Page-level authorization using roles/permissions
  - Prevents unauthorized access automatically

- 🔹 **Dynamic UI Rendering**
  - UI elements adapt based on user roles
  - Controlled via CSS and backend logic

- 🔹 **Database-Driven Architecture**
  - Roles, permissions, and menus stored in SQL
  - Easy to manage and scale


## 🧠 Core Concept

RBAS follows a structured access model:
🔹 User → Role → Permission → Menu Rendering → Page Authorization

- Each user is assigned a role
- Each user has multiple permissions
- Permissions control:
  - Menu visibility
  - Page access
  - UI behavior

## 🏗️ Tech Stack

- ⚙️ **Backend:** .NET Core (Razor Pages)
- 🗄️ **Database:** SQL Server
- 🎨 **Frontend:** HTML, CSS
- 🔐 **Authentication/Authorization:** Custom RBAS logic

## ⚙️ How It Works

- Authorization handled via role-permission mapping
- Menu rendering is database-driven
- Razor Pages enforce page-level access control
- Unauthorized access is blocked at backend level

## 🧪 Dynamic Access Example

The system supports multiple roles with different access levels:

- `Admin`
  - Dashboard, User Management, Settings, Reports
    
- `Manager`
  - Dashboard, Team Overview, Reports
    
- `Sales Manager`
  - Dashboard, Sales Panel, Client Data
  
- `HR Manager`
  - Dashboard, Employee Records, Recruitment
    
- `Executive`
  - Dashboard (limited access)

**`You can add more users access with permissions(CRUD)`**

## 🔄 Behavior

- Menu and submenu are dynamically generated based on assigned role
- Each role sees only the features they are authorized for
- Unauthorized pages are restricted even if accessed via direct URL

## ⚙️ Installation

```bash
git clone https://github.com/iamnomandra/role_base_access_system.git
```

## 🎨 Setup

- **`Entities`**

```bash
[Table("tblMenus")]
public partial class MenuEntity
{
   public int MenuId { get; set; }
   public int ParentId { get; set; }
   public int RoleId { get; set; }
   public int? DepId { get; set; } 
   public string Mname { get; set; } = null!; 
   public string Smname { get; set; } = null!; 
   public string Controller { get; set; } = null!; 
   public string Action { get; set; } = null!; 
   public string Url { get; set; } = null!;
   public string? Icon { get; set; }
   public int? OrderSequence { get; set; }
   public virtual ICollection<PermissionEntity> Permissions { get; set; } = new List<PermissionEntity>();
}

[Table("tblPermissions")]
public partial class PermissionEntity
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int PermissionId { get; set; }
    public int MenuId { get; set; }
    public int UserId { get; set; }
    public bool Create { get; set; }
    public bool Update { get; set; }
    public bool Read { get; set; }
    public bool Delete { get; set; }
    [DisplayName("Is Active")]
    public bool IsActive { get; set; }
    [ForeignKey("MenuId")] 
    public virtual MenuEntity Menu { get; set; } = null!;  
    [ForeignKey("UserId")] 
    public virtual UserEntity User { get; set; } = null!;
}

[Table("tblUsers")]
public class UserEntity 
{
   [Key]
   [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
   [Column("UserId")]
   public int UserId { get; set; }
   [Column("RoleId")]
   [Required, Range(1, int.MaxValue, ErrorMessage = "Please select user role!")]
   public int RoleId { get; set; }

   [ForeignKey("RoleId")] 
   public virtual RoleEntity Role { get; set; } = null!;
   public virtual ICollection<PermissionEntity> Permissions { get; set; } = new List<PermissionEntity>();
}

[Table("tblRoles")]
public partial class RoleEntity
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int RoleId { get; set; }
    public string RoleName { get; set; } = null!; 
    public virtual ICollection<UserEntity>  Users { get; set; } = new List<UserEntity>();
}
```

- **`Permission interface`**
```bash
 List<MenuEntity> Menus(int UserId);
 Task<PermissionEntity> GetPermissions(int? mUserId, string mUrl);
```

- **`Permission repository`**  
```bash
 public List<MenuEntity> Menus(int mUserId)
 {
     var mList = new List<MenuEntity>();
     if (this.mContext != null)
     { 
         if(mUserId > 1)
         {
            
             mList = mContext.tblPermissions.Where(s => s.UserId == mUserId &&  s.IsActive == true).Select(s => s.Menu)
                                           .OrderBy(o => o.OrderSequence).ThenBy(o => o.Smname).AsNoTracking().ToList();
         }
         else
         {
             mList = mContext.tblMenus.OrderBy(o => o.OrderSequence).ThenBy(o=> o.Smname).ToList();
         }
         
     }
     return mList;
 }

 public async Task<PermissionEntity> GetPermissions(int? mUserId, string mUrl)
 { 
      var mModel= new PermissionEntity();
      if (this.mContext != null)
      {
          if (mUserId > 2)
          {
              mModel = await mContext.tblPermissions.Include(p => p.Menu).FirstOrDefaultAsync(s => s.UserId == mUserId && s.Menu.Url.Contains(mUrl)); 
          } 
      }
      return mModel;
 }
```

## ✅ Frontend UI

- **`Layout- On page top`**
```bash
@inject IMenu mMenu  
@inject IPermission mPermission
@inject IWebHostEnvironment _env
@inject IConfiguration mConfiguration
@inject IHttpContextAccessor mAccessor  
.....
string fileInternalName = "";
string versionNumber = "";
string fileModifiedTime = ""; 
@mIApp(IPermission mPermission, IWebHostEnvironment _env, 
           IHttpContextAccessor mAccessor, IHtmlHelper htmlHelper, IConfiguration mConfiguration, 
           ref int? mUserId, ref string menuLoader, ref string fileInternalName, ref string versionNumber, ref string fileModifiedTime)
....
```

- **`Layout- Global permissions`**
```bash
var mUrl = mAccessor.HttpContext.Request.Path.ToString();
var mPermissions = await mPermission.GetPermissions(mUserId, mUrl);
if (mPermissions == null)
{
    mAccessor.HttpContext.Response.Redirect("/AccessDenied/Denied");
}
else
{
    TempData["CreatePermission"] = mPermissions?.Create;
    TempData["UpdatePermission"] = mPermissions?.Update;
    TempData["DetailsPermission"] = mPermissions?.Read;
    TempData["DeletePermission"] = mPermissions?.Delete;
}
```

- **`View- Or use on any view`**

```bash
var mUrl = mAccessor.HttpContext.Request.Path.ToString();
var mPermissions = await mPermission.GetPermissions(mUserId, mUrl);
....
@if (mPermissions?.Create == true)
{
   <a class="btn btn-shadow btn-info" asp-action="Create">
      <span class="btn-icon-wrapper">
         <i class="fa fas fa-regular fa-plus"></i>
      </span>New
   </a>                                   
}
```

- **`Layout- Body tag for menu`**
```bash
<body>
.....

 <ul class="vertical-nav-menu metismenu">
    @Html.Raw(mAccessor.HttpContext.Session.GetString("mMenuString"))
 </ul>

....
</body>
```
## 📜 Screenshots 

- **`Screenshot 01`**

![Screenshot01](https://github.com/iamnomandra/role_base_access_system/blob/main/Screenshot%202026-04-18%20125851.png)

- **`Screenshot 02`**

![Screenshot01](https://github.com/iamnomandra/role_base_access_system/blob/main/Screenshot%202026-04-18%20125717.png)


### 💬 Questions

- Feel free to open an **Issue** or start a **Discussion** on GitHub.
  
## 📜 License

 - MIT License
  
_`Free to use, modify, and enhance in your own projects.`_

