---
title: Extend IdentityUserRole in Dotnet Core 2.1
date: "2018-09-10T22:12:03.284Z"
---


## Problem

dotnet core provides Identity module which is membership system that adds login functionality. It has User module, Role module, and User Role module.

Now I want to extend the User Role (IdentityUserRole) and adding a few foreign keys, but got this error:

>    A key cannot be configured on 'PersonRole' because it is a derived type. The key must be configured on the root type 'IdentityUserRole'. If you did not intend for 'IdentityUserRole' to be included in the model, ensure that it is not included in a DbSet property on your context, referenced in a configuration call to ModelBuilder, or referenced from a navigation property on a type that is included in the model.

This is because if you look into the definition of IdentityUserRole, you'll find out that it already has primary keys: Github link

```
    public class IdentityUserRole<TKey> where TKey : IEquatable<TKey>
    {
        public IdentityUserRole();

        //
        // Summary:
        //     Gets or sets the primary key of the user that is linked to a role.
        public virtual TKey UserId { get; set; }
        //
        // Summary:
        //     Gets or sets the primary key of the role that is linked to the user.
        public virtual TKey RoleId { get; set; }
    }
```

To fix this, we have to override the user role implementation. Now if you see the code, you'll see that we can override it. So I did this:

```
    public class WebCertContext : IdentityDbContext<
        Person,
        Role,
        string,
        IdentityUserClaim<string>,
        PersonRole,
        IdentityUserLogin<string>,
        IdentityRoleClaim<string>,
        IdentityUserToken<string>>
```

The code above means:

- The primary key is using string (TKey equals string type)
- TUser: Person
- TRole: Role
- TUserRole: PersonRole
- For other models, use built in from dotnet core

And this is to override primary keys in PersonRole:

```
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.Entity<PersonRole>(personRole =>
        {
            personRole.HasKey(pr => new
            {
                pr.UserId,
                pr.RoleId,
                pr.ClientId
            });
        }
    }
```

After we override the database context, we can use custom TUserRole, for example like this one:

```
    public class PersonRole : IdentityUserRole<string>
    {
        [Required, StringLength(36)]
        public string ClientId { get; set; }
        [ForeignKey("ClientId")]
        public virtual Client Client { get; set; }
        public virtual Person User { get; set; }
        public virtual Role Role { get; set; }
    }
```

## Summary

Dotnet core has provided powerful feature for User management. We can either use the built in structure or we can override it to our custom need. Which is super awesome! :)

