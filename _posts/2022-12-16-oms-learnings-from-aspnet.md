---
layout: post
title: "ASP.NET 7: I'm in love"
tags: web dotnet aspnet databases
---

One of my latest side projects has been [OpenModServer](https://github.com/jacksonrakena/openmodserver) (which you can try out on [mods.jacksonrakena.com](https://mods.jacksonrakena.com)), a kind of "all-in-one" mod website where users can download mods for many different games. It's a bit of a learning project for me, and I've been using it to practice a lot of skills. I'm going to write about a few of those in this post.

## ASP.NET's comprehensive form validation suite
One of the best features of ASP.NET, ever since those early days, has been its form validation suite. Right from the get-go, it's an absolute breeze to use, and reduces the mental load of writing validation code. It's also very powerful, and can be used to validate forms in a variety of ways.

Let's go to an example:
```html
<div class="mb-3">
    <label asp-for="Changelog" class="form-label"></label>
    <textarea asp-for="Changelog" class="form-control"></textarea>
    <span asp-validation-for="Changelog" class="text-danger"></span>
</div>
```

This chunk of HTML hooks up a field, a label, and a validation span to the `Changelog` field on the page model:
```cs
[BindProperty]
[MaxLength(2048)]
[Required]
public string Changelog { get; set; }
```

Let's go through those attributes, and explain what they mean:
- `[BindProperty]` binds the field to the form. Without this, the field will not be populated when the form is submitted.
- `[MaxLength(2048)]` limits the field to being no more than 2,048 characters.
- `[Required]` means the field must have some content.

You might think that these are done server-side, as they are expressed in the model. But you'd be wrong!
The `asp-for` helper in the `textarea` element actually generates code to tell the browser to validate the field:
```html
<textarea class="form-control" data-val="true" 
data-val-maxlength="The field Changelog must be a string or array type with a maximum length of '2048'."
data-val-maxlength-max="2048"
data-val-required="The Changelog field is required."
id="Changelog"
maxlength="2048"
name="Changelog">
</textarea>
```

So that's cool. The browser does some validation (although the server does it too!), which saves us some server load from genuine users (by preventing this on the client), while preventing bad users from submitting bad data, by verifying it server-side too.


## Just how cool Entity Framework queries are
Entity Framework is a database ORM that is quite heavily abstracted - as in, it uses high-level C# language features and figures out what the developer wants to do, and converts that to raw SQL. It's quite similar to ActiveRecord, from Ruby on Rails, but a fair bit more powerful and, more importantly, customizable.

Let's take for example, this query:
```cs
// Search all mods
var modListings = await _database.ModListings
// Do an INNER JOIN on the users table where the Users.Id == ModListing.AuthorId
            .Include(e => e.Creator)
// Sort by ModListings.DownloadCount, descending
            .OrderByDescending(t => t.DownloadCount)
// Take the first 50 results
            .Take(50)
// And finally, group by the game identifier 
            .GroupBy(d => d.GameIdentifier)
// This method actually sends the request to the database
            .ToListAsync();
```
This is a rather extensive query, involving JOINs and filtering of the output data. One might think from the C# syntax that this is done client-side, but we can see from EF logs that it's all done in the query:
```
SELECT [...]
      FROM (
          SELECT [...]
          FROM mod_listings AS m
          ORDER BY m.download_count DESC
          LIMIT @__p_0
      ) AS t
      INNER JOIN "AspNetUsers" AS a ON t.creator_id = a."Id"
      ORDER BY t.game_identifier
```
This is a rather useful feature. Having to write SQL queries by hand can be a pain (although sometimes necessary to optimize), and switching mental contexts especially so. With EF, you can write your queries in C#, and EF will take care of the rest.

## Having multiple actions on one page
I made the executive decision to use Razor Pages more heavily instead of ASP.NET's traditional Model-View-Controller pattern. Razor Pages somewhat combines the View and Controller parts of those, and allows for a more "page-oriented" approach to web development. This is a bit of a departure from the traditional MVC pattern, but I think it's a good one.

This, however, posits an important problem. How do we have multiple actions on one page?

One answer with Razor Pages is called **page handlers**, and they're pretty neat. They're methods in your Razor Page model, that, when set in a `<button>`, will be called when the button is clicked. ASP.NET automatically hooks up the required route and connects it to your method. Here's an example:
<div class="d-flex flex-column align-items-center p-4">
    <img src="/assets/media/img/2022-12-16-oms-learnings-from-aspnet/page-handler-buttons.png" alt="Page handler buttons" width="800"  />
    <small class="gray pt-2 text-center" style="max-width:400px">A user upload awaiting moderator approval. Note the 'approve' and 'delete' options.</small>
</div>

These buttons are wired up using `asp-page-handler`:
```html
<button asp-page-handler="HandleApproval" type="submit"
    class="btn-sm btn-success">Approve</button>

<button asp-page-handler="HandleDeletion" type="submit"
    class="btn-sm btn-danger">Delete</button>
```

The referenced methods, `HandleApproval`/`HandleDeletion`, are defined in the page model:
```cs
public async Task<IActionResult> OnPostHandleDeletionAsync(string id)
{
    // Parse the ID as a GUID
    if (!Guid.TryParse(id, out var releaseId)) return RedirectToPage("/Files", new { Area = "Admin" });

    // Search for the release
    var release = await _database.ModReleases.FirstOrDefaultAsync(d => d.Id == releaseId);
    if (release == null) return RedirectToPage("/Files", new { Area = "Admin" });

    // Remove it from the database
    _database.ModReleases.Remove(release);

    // Tell the file manager to erase any stored data on disk
    await _fileManagerService.DeleteModReleaseAsync(release);

    // Save the database changes
    await _database.SaveChangesAsync();
    
    // Re-fetch the moderation queue, and re-render the same page
    ModerationQueue = await _database.ModReleases
        .Where(d => d.CurrentStatus == ModReleaseApprovalStatus.Unapproved)
        .ToListAsync();
    
    Releases = await _database.ModReleases.Include(c => c.ModListing).ToListAsync();
    return Page();
}
```

The `string id` parameter you see in the method is passed in through a hidden form field:
```html
<input type="hidden" name="id" value="@release.Id" />
```

(Before you ask about authenticating the user, the page model already does that through `[Authorize("Administrator")]`.)

## Other ASP.NET features that I like
- Full type-safety, even in HTML templates, as CSHTML files are compiled into .NET assembly.
  - This is the big reason I switched to C# over Ruby for full-stack app development. It's really convenient having enforced type-safety, and eliminates an entire class of errors and confusions.
- Best-in-class performance. ASP.NET is a mature framework, and it shows.
- The maturity of NuGet packages. Coming back from the JavaScript world, having a stable, no-nonsense, and well-maintained package ecosystem is a breath of fresh air. Anyone remember `left-pad`? 

## Things I'm not exactly in love with
ASP.NET isn't perfect, and some of the things it does confuses me. Let's go over them:
- Route helpers are not reality-checked. This means if you use an `asp-route` attribute to a non-existent route, you'll get a 404 and some odd behaviour, instead of a compile-time error. This is a bit of a pain, and I wish it was more strict on this. Rider does warn you about non-existent routes, though.
- Although the built-in Identity framework is extensive, the code generator trips me up a few times. There's no way to scaffold all of the templates out so you can customise them, and using the SignInManager/UserManager can be annoying sometimes. Admittedly, this is more of a flaw in managing identity online than it is in ASP.NET. 
- Hot Reload is still nowhere near as good as JavaScript frameworks. Sometimes it breaks randomly and will stay broken for an entire session, and sometimes hot reloading a specific area of code will cause Razor Pages to bug out completely, requiring a full restart. I'll admit that hot reloading a compiled language seems quite difficult, and I appreciate the work that the .NET team does.
- ASP.NET is not as opinionated as frameworks like Rails. This is both a good and bad thing. 

# Conclusion
So those are the big features that I've been enjoying in ASP.NET. After years of React/frontend-heavy development, it's nice to be back in the minimal-to-no-JavaScript land. The page loading much quicker and not being as resource-intensive is a nice change of pace.