# ed-laravel-collection-tips
Laravel Collections: A Simplified Guide with 15 “Chained” Method Examples

## Example 1: Using map with implode

Firstly, let’s see a straightforward example where we chain two methods. Here, our aim is to display permissions separated by an HTML break tag.

### Code Simplified:

```
$role = Role::with('permissions')->first();
$displayPermissions = $role->permissions
    ->map(function($permission) {
        return $permission->name;
    })
    ->implode("<br>");
```

Initially, $role->permissions gives us permission objects. However, we want only the name field from each object. By using map, we create a new collection with just these names. Finally, implode gives us a string like:

```
"manage users<br>manage posts<br>manage comments"
```

## Example 2: Combining map and max

Imagine having an array of scores. Your objective is to determine the highest score from that array.

### Code Simplified:

```
$hazards = [
    'BM-1' => 8,
    'LT-1' => 7,
    // ... [Other values]
    'NoGS' => 0,
    'Not Screened' => 0,
];
$highestScore = Score::all()
    ->map(function($score) use ($hazards) {
        return $hazards[$score->field];
    })
    ->max();
```

Initially, you have the hazards array. After using map, we extract only the numerical values based on the score field. At the end, max looks through these numbers and gives the highest one, which in this example, is:

```
7
```

## Example 3: Using pluck with flatten

Here’s a situation where the initial data, which is from the configuration, presents a multi-level array of settings. Our objective? To extract a simple array of elements.

### Code Simplified:

**Config File (config/setting_fields.php):**

```
return [
    'app' => [
        'title' => 'General',
        // ... [Other details]
        'elements' => [
            [
                'type' => 'text',
                'name' => 'app_name',
                'value' => 'Laravel Starter',
                // ... [Other fields]
            ],
            // ...
        ],
    ],
    'email' => [
        'title' => 'Email',
        // ... [Other details]
        'elements' => [
            [
                'type' => 'email',
                'name' => 'email',
                'value' => 'info@example.com',
                // ... [Other fields]
            ],
        ],
    ],
    'social' => [
        'title' => 'Social Profiles',
        // ... [Other details]
        'elements' => [
            [
                'type' => 'text',
                'name' => 'facebook_url',
                'value' => '#',
                // ... [Other fields]
            ],
        ],
    ],
    // ... [Other sections]
];
```

Extracting Elements:

```
$elementsArray = collect(config('setting_fields'))
    ->pluck('elements')
    ->flatten(1);
```

Here’s what we’re doing step-by-step:

- We convert the array from config('setting_fields') into a Laravel collection.
- We then use pluck('elements') to grab all the elements arrays.
- Finally, flatten(1) is used to transform the multi-dimensional collection into a single-level array.

This process gives us a simplified array of elements without any multi-level complexity.

## Example 4: Using filter + first and map + implode

Let’s walk through a scenario where we want to identify a class’s namespace from a list and then transform that class name into kebab case.

### Simplified Code:

```
$class = 'App\\Base\\Http\\Livewire\\SomeClass';
$availableNamespaces = [
    'App\Base\Http\Livewire',
    'App\Project\Livewire'
];
$foundNamespace = collect($availableNamespaces)
    ->filter(function($namespace) use ($class) {
        return strpos($class, $namespace) !== false;
    })
    ->first();
$kebabCasedNamespace = collect(explode('.', str_replace(['/', '\\'], '.', $foundNamespace)))
    ->map([Str::class, 'kebab'])
    ->implode('.');
```

### Step-by-step Explanation:

We start with a specific class and a list of potential namespaces.

Using filter(), we sift through the namespaces to find a match for our class. We then grab the matched namespace with first().

After this step, our $foundNamespace would be:

```
"App\Base\Http\Livewire"
```

We then proceed to transform the namespace to kebab case:

- Replace slashes with dots to segment the namespace.
- Break the segmented namespace into a collection.
- Use map() to apply the kebab casing.

Here’s a neat trick: instead of writing out a full callback for map(), you can point it directly to a method, like Str::kebab.

Finally using implode(), we join our kebab-cased segments back into a single string. Our result would be:

```
"app.base.http.livewire"
```

So, with just a few collection methods, we’ve successfully found a namespace and transformed it into kebab case!

## Example 5: Combining push, map, and implode

Let’s delve into a scenario with three collection methods. Imagine you’re running a Twitter giveaway command in Laravel’s Artisan, and you want to exclude specific users.

### Simplified Code:

```
$excludedNames = collect($this->option('exclude'))
    ->push('povilaskorop', 'dailylaravel')
    ->map(function (string $name): string {
        return str_replace('@', '', $name);
    })
    ->implode(', ');
```

Command Usage:

```
php artisan twitter:giveaway --exclude=someuser --exclude=@otheruser
```

### Explanation in Steps:

Starting Point: The command lets you exclude users with the --exclude option. The $this->option('exclude') would give an array of user names you want to exclude.

Initially, after turning it into a collection, it might look like:

```
["someuser", "@otheruser"]
```

Using push(): We want to add specific usernames to the exclusion list. By using push(), we add 'povilaskorop' and 'dailylaravel' to our collection.

After this step, the collection might look like:

```
["someuser", "@otheruser", "povilaskorop", "dailylaravel"]
```

Applying map(): We want our usernames to be consistent. The '@' symbol is removed to ensure uniformity. The map() method helps us iterate through the list and remove any '@' from the usernames.

The collection, after using map, would be:

```
["someuser", "otheruser", "povilaskorop", "dailylaravel"]
```

Finishing with implode(): For easy display, we want to join the names together into a single string. implode() helps us achieve that, separating each username with a comma.

Thus, our final result would be:

```
"someuser, otheruser, povilaskorop, dailylaravel"
```

With these three methods, we’ve tidied up and formatted our list of excluded Twitter usernames.


## Example 6: Combining filter, map, and implode

In this scenario, consider having a User model with multiple social media profile links. Our objective is to display these links as clickable hyperlinks and omit any that are empty.

### Simplified Code:

```
$socialProfileLinks = collect([
    'Twitter' => $user->link_twitter,
    'Facebook' => $user->link_facebook,
    'Instagram' => $user->link_instagram,
])
->filter()
->map(function ($link, $platform) {
    return '<a href="' . $link . '">' . $platform . '</a>';
})
->implode(' | ');
```

### Step-by-step Explanation:

Starting Point: We begin by creating a collection of social media profiles.

Initially, it might look like:

```
['Twitter' => 'https://twitter.com/username', 'Facebook' => '', 'Instagram' => 'https://instagram.com/username']
```

Using filter(): Some profile links might be empty, so we use filter() to remov0e them. By default, filter() will omit empty values.

After filtering, our collection might appear as:

```
['Twitter' => 'https://twitter.com/username', 'Instagram' => 'https://instagram.com/username']
```

Applying map(): Now, we want to convert the URLs into clickable links. The map() method lets us iterate over each link, wrapping them in HTML anchor tags.

Post mapping, the collection would transform into:

```
["<a href='https://twitter.com/username'>Twitter</a>", "<a href='https://instagram.com/username'>Instagram</a>"]
```

Finishing with implode(): To make it presentable, we join the individual links into a single string using implode(), with each link separated by a vertical bar ('|').

The final result would be:

```
"<a href='https://twitter.com/username'>Twitter</a> | <a href='https://instagram.com/username'>Instagram</a>"
```

With this flow, you can effortlessly format and display a user’s active social media profiles in your Laravel blade files.

## Example 7: Chain of reject, reject, and each

Let’s dive into a situation where we need to filter a list of models based on certain conditions and then perform an action on the remaining models.

### Simplified Code:

```
Repository::query()
    ->with('owner')
    ->get()
    ->reject(function (Repository $repository) {
        return $repository->owner instanceof User && $repository->owner->github_access_token === null;
    })
    ->reject(function (Repository $repository) {
        return $repository->owner instanceof Organization && !$repository->owner->members()->whereIsRegistered()->exists();
    })
    ->each(function (Repository $repository) {
        UpdateRepositoryDetails::dispatch($repository);
    });
```

### Step-by-step Explanation:

Starting Point: We’re pulling a list of repositories with their respective owners.

At the beginning, it might look like:

```
[Repository1 with User without GitHub token, Repository2 with Organization having registered members, ...]
```

First reject(): We want to exclude repositories where the owner is a User and doesn't have a GitHub access token.

After the first rejection, our list might become:

```
[Repository2 with Organization having registered members, ...]
```

Second reject(): Next, we want to further filter out repositories where the owner is an Organization and doesn't have any registered members.Following this second rejection, our list might narrow down further.

Using each(): With the filtered list, we now loop through each repository to update its details.

## Example 8: Using mapWithKeys, forget, and filter

Here, we’ll navigate through a scenario where you possess log files and wish to identify those older than a specific duration.

### Simplified Code:

```
$files = Storage::disk("logs")->allFiles();
$filteredLogFiles = collect($files)
    ->mapWithKeys(function ($file) {
        $matches = [];
        $isMatch = preg_match("/^laravel\-(.*)\.log$/i", $file, $matches);
        $date = ($isMatch && count($matches) > 1) ? $matches[1] : "";
        return [$date => $file];
    })
    ->forget("")
    ->filter(function ($filename, $date) use ($thresholdDate) {
        try {
            $logDate = Carbon::parse($date);
        } catch (\Exception $e) {
            return true;
        }
        return $logDate->isBefore($thresholdDate);
    });
```

### Step-by-step Explanation:

Starting Point: First, we pull a list of all log files using Laravel’s storage system.

Initially, our collection might look like:

```
[laravel-2022-01-01.log, laravel-2022-01-02.log, ...]
```

Using mapWithKeys(): We wish to associate each file with its date. The purpose here is to return a key-value array, where the key is the log's date, and the value is its filename.

Post this operation, our collection might transform into:

```
[2022-01-01 => laravel-2022-01-01.log, ...]
```

Applying forget(): Some files might not follow the standard naming and won't have a date. We exclude these using forget().

After applying forget(), our list becomes cleaner:

```
[2022-01-01 => laravel-2022-01-01.log, 2022-01-02 => laravel-2022-01-02.log, ...]
```

Finishing with filter(): We now sift through the files and retain only those older than our specified threshold date.

Our final list, after filtering, might be a narrowed down set of older log files.

This sequence of collection methods allows you to efficiently pinpoint and manage outdated log files in your Laravel application.

## Example 9: Using map, filter, and each

Let’s explore a scenario typical to social platforms. Imagine you have a comment where users are mentioned using the @username format. From this, we aim to identify the mentioned users and notify them.

### Simplified Code:

```
$comment = Comment::first();
collect($comment->mentionedUsers())
    ->map(function ($username) {
        return User::where('name', $username)->first();
    })
    ->filter()
    ->each(function ($user) use ($comment) {
        $user->notify(new YouWereMentionedNotification($comment));
    });
```

### Step-by-step Explanation:

Starting Point: We begin with a comment which might look like:

```
"I mention the @First user and the @Second user and @Third non-existing one."
```

Extracting Usernames: We need a method to pull out usernames mentioned in the comment. The mentionedUsers() function accomplishes this using a regex pattern.

```
public function mentionedUsers() {
    preg_match_all('/@([\w\-]+)/', $this->description, $matches);
    return $matches[1];
}
```

This would give us a collection like:

```
["First", "Second", "Third"]
```

Using map(): Next, for each username, we attempt to fetch the corresponding user from the database.

Post mapping, our collection might have:

```
[UserObject (First), UserObject (Second), null (for "Third")]
```

Applying filter(): As some usernames might not correspond to actual users, we filter out any null results.

Our filtered collection would then look like:

```
[UserObject (First), UserObject (Second)]
```

Finishing with each(): With the verified list of users, we loop through each and send them a notification about the mention.

By chaining these methods, you can efficiently extract, verify, and notify mentioned users in your Laravel application.

## Example 10: Using map, filter, and implode

In this illustration, imagine you’re working with a list of categories. These categories have a hierarchical structure, and our aim is to obtain the slugs of categories in the active language, creating a structured URL path.

### Simplified Code:

```
$activeLanguage = 'en';
$slugsPath = Category::all()
    ->map(function ($category) use ($activeLanguage) {
        return $category->getSlug($activeLanguage);
    })
    ->filter()
    ->implode('/');
```

### Step-by-step Explanation:

- Starting Point: We fetch all categories.
- Using map(): Each category has a method getSlug() that gets its slug based on the provided language.

```
public function getSlug($language = null) {
    $activeSlug = $this->getActiveSlug($language);
if ($activeSlug) {
        return $activeSlug->slug;
    }
    if (config('translatable.use_property_fallback', false)) {
        $fallbackSlug = $this->getFallbackActiveSlug();
        return $fallbackSlug ? $fallbackSlug->slug : "";
    }
    return "";
}
public function getActiveSlug($language = null) {
    return $this->slugs->first(function ($slug) use ($language) {
        return $slug->active && $slug->locale === ($language ?? app()->getLocale());
    });
}
public function getFallbackActiveSlug() {
    return $this->slugs->first(function ($slug) {
        return $slug->active && $slug->locale === config('translatable.fallback_locale');
    });
}
```

This set of methods ensures that for each category, we either get its slug in the desired language, its fallback slug, or an empty string.

After mapping, our collection might look like:

```
["first-category", "second-category", "", "third-category", ...]
```

Applying filter(): Some categories might not have a corresponding slug, resulting in empty strings. We use filter() to exclude these.

Our collection, post filtering, might appear as:

```
["first-category", "second-category", "third-category"]
```

Finishing with implode(): For the final touch, we join the slugs to craft a URL path. implode('/') links them with slashes.

The outcome would be:

```
"first-category/second-category/third-category"
```

Using this sequence, you can conveniently construct a structured URL path from categories in your Laravel application.

## Example 11: Chain of filter, filter, and each

In this illustration, we’re working with a set of Posts. The goal is to identify those that are categorized as Tweets, further filter out those that lack an external URL, and then populate their external URLs if they contain a link to twitter.com.

### Simplified Code:

```
Post::all()
    ->filter->isTweet()
    ->filter(function (Post $post) {
        return empty($post->external_url);
    })
    ->each(function (Post $post) {
        preg_match('/(?=https:\/\/twitter.com\/).+?(?=")/', $post->text, $matches);
        if (count($matches) > 0) {
            $post->external_url = $matches[0];
            $post->save();
        }
    });
```

### Step-by-step Explanation:

Starting Point: We begin by fetching all Posts.

First filter - Using isTweet(): We aim to isolate Posts that are Tweets. Laravel collections allow us to use methods as properties, so we can use filter->isTweet().

The isTweet() method is simple:

```
public function isTweet(): bool {
    return $this->getType() === 'tweet';
}
```

This relies on getType(), which determines the type of the post:

```
public function getType(): string {
    if ($this->hasTag('tweet')) {
        return 'tweet';
    }
    if ($this->original_content) {
        return 'original';
    }
    return 'link';
}
```

The hasTag() method checks if a post has a specific tag:

```
public function hasTag(string $tagName): bool {
    return $this->tags->contains(fn (Tag $tag) => $tag->name === $tagName);
}
```

After applying isTweet(), our collection retains:

```
[Tweet1, Tweet2, ...]
```

Second filter: Now, we filter out Tweets that lack an external URL. Our collection narrows further.

Using each(): For each of the remaining Tweets, we inspect if their text contains a link to twitter.com. If so, we update the external_url property and save the post.

By chaining these collection methods, you can efficiently process and update specific Tweets in your Laravel application.

## Example 12: Using unique, filter, map, and values

In this scenario, we’re working with events. Each event has attributes like a message, status, and subject. The challenge is to extract unique messages, ensure they have a subject, and then transform the data into a new structure.

### Simplified Code:

```
$eventsCollection = Event::all();
$processedEvents = $eventsCollection
    ->unique(fn ($event) => $event->message)
    ->filter(fn ($event) => !is_null($event->subject))
    ->map(fn ($event) => $this->extractData($event))
    ->values();
```

### Step-by-step Explanation:

Starting Point: We first fetch all events from the database.

Applying unique(): There might be events with identical messages. Using the unique() method, we weed out these duplicates based on their message attribute.

For example, if events with ID 1 and 2 share the same message, one will be excluded.

After this step, our collection could look like:

```
[Event1, Event3, Event4, ...]
```

Next, filter(): We want our final list to only include events that possess a subject. The filter() method helps us ensure that only events with a subject remain in our collection.

Post filtering, the list might further reduce:

```
[Event1, Event4, ...]
```

Then, map(): Now, it's time to modify our data. For each event in our list, we want to transform it into a different structure. The extractData() method (not provided in the original content) presumably performs this transformation.

This step might reshape each event, making the collection look like:

```
[TransformedEvent1, TransformedEvent4, ...]
```

Ending with values(): After several operations, our collection's keys might be out of sequence. The values() method resets these keys, ensuring they are sequential again.

By chaining these collection methods, we efficiently refine and transform our event data in the Laravel application.

## Example 13: Using map, mapToGroups, map, and each

In this illustration, we aim to track the latest versions of Laravel and notify users about recent updates.

### Simplified Code:

```
$githubVersions = collect([
    ['name' => 'v9.19.0'],
    ['name' => 'v8.83.18'],
    // ... other versions ...
]);
$processedVersions = $githubVersions
    ->map(fn ($version) => explode('.', ltrim($version['name'], 'v')))
    ->mapToGroups(fn ($numbers) => [$numbers[0] => $numbers])
    ->map(function ($group) {
        // Logic for sorting and picking the latest version
    })
    ->each(function ($versionDetails) {
        // Database operations and info logging
    });
```

### Step-by-step Explanation:

Starting Point: We begin with versions sourced from GitHub.

First map(): We simplify the version by removing the 'v' prefix and splitting it into major, minor, and patch numbers.

Using mapToGroups(): We categorize the versions. If the major version is less than 6, versions are grouped by both major and minor numbers. For versions 6 and above, they're grouped just by major numbers.

Second map(): In this step, we refine each group to pinpoint the most recent version. This involves sorting based on minor and patch versions.

Lastly, each(): For every identified version, we undertake a set of database operations. If a version (based on major and minor) already exists in our database, we just update its patch number. If it doesn’t exist, we create a new entry. We also log updates and creations for tracking purposes.

Through these method chains, we seamlessly process the latest Laravel versions and update our database accordingly.

## Example 14: Using map, flatten, map, and filter

In this scenario, we want to sift through two directories where developers can place Livewire components. Our goal is to produce a refined list of these components, represented by their path names.

### Simplified Code:

```
$directories = collect([
    "/Users/Povilas/Sites/project3/app/Base/Http/Livewire/",
    "/Users/Povilas/Sites/project3/app/Project/Livewire/"
]);
$componentList = $directories
    ->map(fn ($dir) => (new Filesystem())->allFiles($dir))
    ->flatten()
    ->map(function (\SplFileInfo $file) {
        return app()->getNamespace().str_replace(
            ['/', '.php'],
            ['\', ''],
            Str::after($file->getPathname(), app_path().'/')
        );
    })
    ->filter(function (string $className) {
        return is_subclass_of($className, Component::class) &&
               ! (new \ReflectionClass($className))->isAbstract();
    });
```

### Step-by-step Explanation:

Starting Point: We have two folders where Livewire components might be placed.

First map(): For each directory, we fetch all the files within it.

Using flatten(): Since the above step might produce nested arrays of files, flatten() consolidates them into a single, flat list.

Second map(): Here, we convert each file's path into a fully qualified class name by replacing slashes with backslashes, and stripping off the .php extension.

Lastly, filter(): We wrap up by retaining only those classes that extend the Livewire Component class and are not abstract.

Following this chain of methods, we end up with a concise list of usable Livewire components from the given directories.

## Example 15: Using mapWithKeys, each, reject, filter, and all

In this scenario, our objective is to sift through a collection of composer packages, singling out only those related to Laravel. The key info resides in $package['extra']['laravel'].

### Simplified Code:

```
if ($filesystem->exists($composerPath = base_path() . '/vendor/composer/installed.json')) {
    $allPackages = json_decode($filesystem->get($composerPath), true);
}
$laravelPackages = collect($allPackages['packages'])
    ->mapWithKeys(fn ($package) => [$this->format($package['name']) => $package['extra']['laravel'] ?? []])
    ->each(fn ($config) => $ignore = array_merge($ignore, $config['dont-discover'] ?? []))
    ->reject(fn ($config, $packageName) => in_array($packageName, $ignore))
    ->filter()
    ->all();
```

### Step-by-step Explanation:

Starting Point: We begin by fetching a list of all installed composer packages.

Using mapWithKeys(): We trim down the package data, retaining only the package name and its associated Laravel-related settings (['extra']['laravel']).

Using each(): For each package, if it has a dont-discover configuration, we merge its value into our ignore list. This gives us a clear list of packages we want to exclude.

Then, reject(): Next, we sift out any packages present in our ignore list.

Applying filter(): To further clean our list, we use filter() to remove any packages that have empty Laravel-related settings.

Finally, all(): We finish off by converting our refined collection back into a plain PHP array.

In the end, we’ve neatly filtered out just the Laravel-related packages we’re interested in, leaving behind the chaff.

