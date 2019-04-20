# A Brief Primer Updating from v11 to v12

After a long time in development, Discord.js v12 is nearing a stable release, meaning it's time to update from v11 to get new features for your bots!  However, with those new features comes a lot of changes to the library that will break code written for v11.  This guide will serve as a handy reference for updating your code, covering the most commonly-used methods that have been changed, new topics such as partials and internal sharding, and will also include a comprehensive list of the method and property changes at the end.

## Before You Start

v12 requires Node 10.x or higher to  use, so make sure you're up-to-date.  To check your Node version, use `node -v` in your terminal or command prompt, and if it's not high enough, update it!  There are many resources online to help you get up-to-date.

For now, you do need Git installed and added to your PATH environment, so ensure that's done as well - again, guides are available online for a wide variety of operating systems.  Once you have Node up-to-date and Git installed, you can install v12 by running `npm install discordjs/discord.js` in your terminal or command prompt for text-only use, or `npm install discordjs/discord.js node-opus` for voice support.

## Commonly Used Methods That Changed

* All section headers are named in the following convention: `Class#methodOrProperty`.
* The use of parenthesis designates optional inclusion. For example, `Channel#fetch(Pinned)Message(s)` means that this section will include changes for `Channel#fetchPinnedMessages`, `Channel#fetchMessages`, and `Channel#fetchMessage`.
* The use of asterisks designates a wildcard. For example, `Channel#send***` means that this section will include changes for `Channel#sendMessage`, `Channel#sendFile`, `Channel#sendEmbed`, and so forth.

### Collection

#### Collection#exists

`collection.exists()` was removed entirely, `collection.some()` should be used to check if an element exists in the collection that satisfies the provided value.

```diff
- client.users.exists('username', 'Bob');
+ client.users.some(user => user.username === 'Bob');
```

#### Collection#filterArray

`collection.filterArray()` was removed entirely, as it was just a helper method for `collection.filter().array()` and most of the time converting a collection to an array is an unnecessary step.

#### Collection#find

`collection.find('property', value)` has been removed entirely, and `collection.find()` only accepts a function in v12.

```diff
- client.users.find('username', 'Bob');
+ client.users.find(user => user.username === 'Bob');
```

#### Collection#findAll

`collection.findAll()` was removed entirely as it just duplicated `collection.filterArray()`'s results.

### Fetch

Some methods that retrieve uncached data have been changed, transformed in the shape of a DataStore.

```diff
- client.fetchUser('123456789012345678');
+ client.users.fetch('123456789012345678');

- guild.fetchMember('123456789012345678');
+ guild.members.fetch('123456789012345678');

- guild.fetchMembers();
+ guild.members.fetch();

- textChannel.fetchMessage('123456789012345678');
+ textChannel.messages.fetch('123456789012345678');

- textChannel.fetchMessages({ limit: 10 });
+ textChannel.messages.fetch({ limit: 10 });

- textChannel.fetchPinnedMessages();
+ textChannel.messages.fetchPinned();
```

### Send

All the `.send***()` methods have been removed in favor of one general `.send()` method.

```diff
- channel.sendMessage('Hey!');
+ channel.send('Hey!');

- channel.sendEmbed(embedVariable);
+ channel.send(embedVariable);
+ channel.send({ embed: embedVariable });
```

`channel.send(embedVariable)` will only work if that variable is an instance of the `MessageEmbed` class; object literals won't give you the expected result unless your embed data is inside an `embed` key.

```diff
- channel.sendCode('js', 'const version = 11;');
+ channel.send('const version = 12;', { code: 'js' });

- channel.sendFile('./file.png');
- channel.sendFiles(['./file-one.png', './file-two.png']);
+ channel.send({
  files: [{
    attachment: 'entire/path/to/file.jpg',
    name: 'file.jpg'
  }]
+ channel.send({
  files: ['https://cdn.discordapp.com/icons/222078108977594368/6e1019b3179d71046e463a75915e7244.png?size=2048']
});
```

### Roles

The `GuildMember.roles` Collection has been changed to a DataStore in v12, so a lot of the associated methods for interacting with a member's roles have changed as well.  They're no longer on the GuildMember object itself, but instead now on the `GuildMemberRoleStore` DataStore.

```diff
- guildMember.addRole('123456789012345678');
- guildMember.addRoles(['123456789012345678', '098765432109876543']);
+ guildMember.roles.add('123456789012345678');
+ guildMember.roles.add(['123456789012345678', '098765432109876543']);

- guildMember.removeRole('123456789012345678');
- guildMember.removeRoles(['123456789012345678', '098765432109876543']);
+ guildMember.roles.remove('123456789012345678');
+ guildMember.roles.remove(['123456789012345678', '098765432109876543']);

- guildMember.setRoles(['123456789012345678', '098765432109876543']);
+ guildMember.roles.set(['123456789012345678', '098765432109876543']);
```

In addition, the GuildMember properties related to roles have also been moved to the `GuildMemberRoleStore` DataStore.

```diff
- guildMember.colorRole;
+ guildMember.roles.color;

- guildMember.highestRole;
+ guildMember.roles.highest;

- guildMember.hoistRole;
+ guildMember.roles.hoist;
```

### Ban and Unban

The method to ban members and users have been moved to the `GuildMemberStore` Data Store.

```diff
- guildMember.ban();
- guild.ban('123456789012345678');
+ guild.members.ban('123456789012345678');

- guild.unban('123456789012345678');
+ guild.members.unban('123456789012345678');
```

### Image URLs

Some image-related properties like `user.avatarURL` are now a method in v12, so that you can apply some options to them, eg. to affect their display size.

```diff
- user.avatarURL;
+ user.avatarURL();

- user.displayAvatarURL;
+ user.displayAvatarURL();

- guild.iconURL;
+ guild.iconURL();

- guild.splashURL;
+ guild.splashURL();
```

### RichEmbed Constructor

The RichEmbed constructor has been removed and now the `MessageEmbed` constructor is used.  It is largely the same to use, the only difference being the removal of `RichEmbed.attachFile()` - `MessageEmbed.attachFiles()` accepts a single file as a parameter as well.

### String Concatenation

v12 has changed how string concatenation works with stringifying objects.  The `valueOf` any data structure will return its id, which affects how it behaves in strings, eg. using an object for a mention.  In v11, you used to be able to use `channel.send(userObject + ' has joined!')` and it would automatically stringify the object and it would become the mention (`@user has joined!`), but in v12, it will now send a message that says `123456789012345678 has joined` instead.  Using template literals (\`\`) will still return the mention, however.

```diff
- channel.send(userObject + ' has joined!')
+ channel.send(`${userObject} has joined!`)
```

### User Account-Only Methods

All user account-only methods have been removed, as they are no longer publicly accessible from the API.

---
<p class="danger">This stuff should keep getting shoved to the bottom, with the commonly-used methods that are changed, as well as topic overviews added before it.</p>

The section headers for breaking changes will be named after the v11 classes/methods/properties and will be in alphabetical order, so that you can easily find what you're looking for. The section headers for additions will be named after the v12 classes/methods/properties, to reflect their current syntax appropriately.

"Difference" codeblocks will be used to display the old methods vs the newer ones—the red being what's been removed and the green being its replacement. Some bits may have more than one version of being handled. Regular JavaScript syntax codeblocks will be used to display the additions. 

<p class="danger">While this list has been carefully crafted, it may be incomplete! If you notice pieces of missing or inaccurate data, we encourage you to [submit a pull request](https://github.com/Danktuary/Making-Bots-with-Discord.js)!</p>

## Breaking Changes and Deletions

<p class="danger">This next bit is for me (Sanc) to keep track of the classes I've gone through and checked for breaking changes. Remove before making the PR.</p>

<p class = "danger">Whatever happened here: https://github.com/discordjs/discord.js/pull/2765/files needs to be added as well as internal sharding and partials</p>

* ClientUser
* DiscordAPIError
* DMChannel
* GroupDMChannel
* Guild
* GuildAuditLogs
* GuildAuditLogsEntry
* GuildChannel
* GuildMember
* Invite
* Message
* MessageAttachment
* MessageMentions
* MessageReaction
* OAuth2Application
* PermissionOverwrites

TODO LATER CUZ I DON'T WANNA TOUCH THAT SHIT RN:

* MessageCollector
* MessageEmbed***

Stuff Mark has gone through and updated - anything under Additions at the end will still likely need descriptions:

* Activity
* ActivityFlags
* APIMessage
* Base
* BaseClient
* BitField
* BroadcastDispatcher
* ClientApplication
* Client
* ClientOptions
* ClientUser
* Channel (changed send/fetch to TextChannel)
* Collector
* CollectorOptions
* DMChannel
* Emoji
* EvaluatedPermissions
* Game
* GroupDMChannel
* Guild
* GuildChannel
* GuildMember
* HTTPError
* Integration
* Invite
* Message
* MessageReaction
* OAuth2Application
* PartialGuild
* PartialGuildChannel
* PermissionOverwrite
* Permissions
* Presence
* ReactionCollector
* ReactionEmoji
* RichEmbed
* RichPresenceAssets
* Role



Stuff Mark didn't do:

* MessageAttachment

### Dependencies

#### Snekfetch

Please note that `snekfetch` has been removed as a dependency, and has been replaced by `node-fetch`.  `snekfetch` has been deprecated by its developer and is no longer maintained.

### Attachment

The `Attachment` class has been removed in favor of the `MessageAttachment` class.

### Client

#### Client#fetchUser

`client.fetchUser()` has been removed and transformed in the shape of a DataStore.

```diff
- client.fetchUser('123456789012345678');
+ client.users.fetch('123456789012345678');
```

#### Client#broadcasts

`client.broadcasts` has been removed and is now in the `ClientVoiceManager` class.

```diff
- client.broadcasts;
+ client.voice.broadcasts;
```

#### Client#browser

`client.browser` has been changed to be an internal constant and is no longer available publicly.

#### Client#channels

`client.channels` has been changed from a Collection to a DataStore.

#### Client#clientUserGuildSettingsUpdate

The `client.clientUserGuildSettingsUpdate` event has been removed entirely, along with all other user account-only properties and methods.

#### Client#clientUserSettingsUpdate

The `client.clientUserSettingsUpdate` event has been removed entirely, along with all other user account-only properties and methods.

#### Client#disconnect

The `client.disconnect` event has been removed in favor of the `client.shardDisconnected` event to make use of internal sharding.

```diff
- client.on('disconnect', event => {});
+ client.on('shardDisconnected', (event, shardID) => {});
```

#### Client#emojis

`client.emojis` has been changed from a Collection to a DataStore.

#### Client#guilds

`client.guilds` has been changed from a Collection to a DataStore.

#### Client#ping

`client.ping` has been moved to the WebSocketManager under `client.ws.ping`

```diff
- client.ping
+ client.ws.ping
```

#### Client#pings

`client.pings` has been moved to the `WebSocketShard` class to make use of internal sharding.  The `Client` class has a `Collection` of `WebSocketShard`s available via `client.ws.shards`; alternatively, the `WebSocketShard` can be found as a property of other structures, eg `guild.shard`.

```diff
- client.pings;
+ guild.shard.pings;
```

#### Client#presences

`client.presences` has been removed to reduce extraneous getters.

#### Client#reconnecting

The `client.reconnecting` event has been removed in favor of the `client.shardReconnecting` event to make use of internal sharding.

```diff
- client.on('reconnecting', () => console.log('Successfully reconnected.'));
+ client.on('shardReconnecting', id => console.log(`Shard with ID ${id} reconnected.`));
```

#### Client#resume

The `client.resume` event has been removed in favor of the `client.shardResumed` event to make use of internal sharding.

```diff
- client.on('resume', replayed => console.log(`Resumed connection and replayed ${replayed} events.`));
+ client.on('shardResumed', (replayed, shardID) => console.log(`Shard ID ${shardID} resumed connection and replayed ${replayed} events.`));
```

#### Client#status

The `client.status` property has been removed and is now in the `WebSocketManager` class.  In addition, it is no longer a getter.

```diff
- client.status;
+ client.ws.status;
```

#### Client#syncGuilds

`client.syncGuilds()` has been removed entirely, along with all other user account-only properties and methods.

#### Client#typingStop

The `client.typingStop` event has been removed entirely, as it was an event created by the library and not an actual Discord WebSocket event.

#### Client#userNoteUpdate

The `client.userNoteUpdate` event has been removed entirely, along with all other user account-only properties and methods.

#### Client#users

`client.users` has been changed from a Collection to a DataStore.

#### Client#voiceConnections

`client.voiceConnections` has been removed and is now in the `ClientVoiceManager` class.  In addition, the `Collection` is no longer a getter.

```diff
- client.voiceConnections;
+ client.voice.connections;
```

#### Client#voiceStateUpdate

The `client.voiceStateUpdate` event now returns `oldState` and `newState` representing the `VoiceState` of the member before and after the update, as opposed to the member itself.

```diff
- client.on('voiceStateUpdate', (oldMember, newMember) => console.log(oldMember));
+ client.on('voiceStateUpdate', (oldState, newState) => console.log(oldState));
```

### ClientOptions

There have been several changes made to the `ClientOptions` object located in `client#options`.

#### ClientOptions#apiRequestMethod

`clientOptions.apiRequestMethod` has been made sequential and is used internally.

#### ClientOptions#shardId

`clientOptions.shardId` has been changed to `clientOptions.shards` and now also accepts an array of numbers.

```diff
- options.shardId: 1
+ options.shards: [1, 2, 3]
```

#### ClientOptions#shards

`clientOptions.shards` has been removed and is functionally equivalent to `clientOptions.totalShardCount` on v12.

#### ClientOptions#sync

`clientOptions.sync` has been removed entirely, along with all other user account-only properties and methods.

### ClientUser

#### ClientUser#acceptInvite

`clientUser.acceptInvite()` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#addFriend

`clientUser.addFriend()` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#avatarURL

`clientUser.avatarURL` is now a method, as opposed to a property. It also allows you to determine the file format and size to return.

```diff
- clientUser.avatarURL;
+ clientUser.avatarURL();
+ clientUser.avatarURL({ format: 'png', size: 1024 });
```

#### ClientUser#block

`clientUser.block()` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#blocked

`clientUser.blocked` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#createGuild

`clientUser.createGuild()` has been transformed in the shape of a DataStore.  In addition, the second and third parameters in `clientUser.createGuild()` have been changed/removed, leaving it with a total of two parameters. The `region` and `icon` parameters from v11 have been merged into an object as the second parameter.

```diff
- clientUser.createGuild('New server', 'us-east', './path/to/file.png');
+ clientUser.guilds.create('New server', { region: 'us-east', icon: './path/to/file.png' });
```

#### ClientUser#displayAvatarURL

`clientUser.displayAvatarURL` is now a method, as opposed to a property. It also allows you to determine the file format and size to return.

```diff
- clientUser.displayAvatarURL;
+ clientUser.displayAvatarURL();
+ clientUser.displayAvatarURL({ format: 'png', size: 1024 });
```

#### ClientUser#email

`clientUser.email` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#fetchMentions

`clientUser.fetchMentions()` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#fetchProfile

`clientUser.fetchProfile()` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#friends

`clientUser.friends` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#guildSettings

`clientUser.guildSettings` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#mfaEnabled

`clientUser.mfaEnabled` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#mobile

`clientUser.mobile` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#note

`clientUser.note` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#notes

`clientUser.notes` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#premium

`clientUser.premium` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#removeFriend

`clientUser.removeFriend()` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#send\*\*\*

Just like the `TextChannel#send***` methods, all the `.send***()` methods have been removed in favor of one general `.send()` method. Read through the [TextChannel#send\*\*\*](/additional-info/changes-in-v12?id=channelsend) section for more information.

#### ClientUser#setGame

`clientUser.setGame()` has been changed to `clientUser.setActivity()`. The second parameter is no longer for providing a streaming URL, but rather an object that allows you to provide the URL and activity type.

```diff
- clientUser.setGame('with my bot friends!');
+ clientUser.setActivity('with my bot friends!');

- clientUser.setGame('with my bot friends!', 'https://twitch.tv/your/stream/here');
+ clientUser.setActivity('with my bot friends!', { url: 'https://twitch.tv/your/stream/here', type: 'STREAMING' });
```

#### ClientUser#setNote

`clientUser.setNote()` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#setPassword

`clientUser.setPassword()` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#settings

`clentUser.settings` has been removed entirely, along with all other user account-only properties and methods.

#### ClientUser#unblock

`clientUser.unblock()` has been removed entirely, along with all other user account-only properties and methods.

### ClientUserChannelOverride

The `ClientUserChannelOverride` class has been removed entirely, along with all other user account-only properties and methods.

### ClientUserGuildSettings

The `ClientUserGuildSettings` class has been removed entirely, along with all other user account-only properties and methods.

### ClientUserSettings

The `ClientUserSettings` class has been removed entirely, along with all other user account-only properties and methods.

### ClientUserChannelOverride

The `ClientUserChannelOverride` class has been removed entirely.

### ClientUserGuildSettings

The `ClientUserGuildSettings` class has been removed entirely.

### ClientUserSettings

The `ClientUserSettings` class has been removed entirely.

### Collection

#### Collection#find/findKey

Both methods will now return `undefined` if nothing is found.

#### Collection#deleteAll

`collection.deleteAll()` has been removed in favor of map's default `clear()` method.

```diff
- roles.deleteAll();
+ roles.clear();
```

#### Collection#exists

`collection.exists()` has been removed entirely in favor of `collection.some()`

```diff
- client.users.exists('username', 'Bob');
+ client.users.some(user => user.username === 'Bob');
```

#### Collection#filterArray

`collection.filterArray()` has been removed completely.

#### Collection#findAll

`collection.findAll()` has been removed completely as the same functionality can be obtained through `collection.filter()`.

#### Collection#first/firstKey/last/lastKey/random/randomKey

The `amount` parameter of these methods now allows a negative number which will start the query from the end of the collection instead of the start.

#### Collection#tap

`collection.tap` runs a specific function over the collection instead of mimicking `<array>.forEach()`, this functionality was moved to `collection.each()`. 

### Collector

#### Collector#cleanup

`collector.cleanup()` has been removed entirely.

#### Collector#handle

`collector.handle()` has been changed to `collector.handleCollect()`.

#### Collector#postCheck

`collector.postCheck()` has been changed to `collector.checkEnd()`.

### DMChannel

#### DMChannel#acknowledge

`dmChannel.acknowledge()` has been removed entirely, along with all other user account-only properties and methods.

#### DMChannel#createCollector

`dmChannel.createCollector()` has been removed in favor of `dmChannel.createMessageCollector()`.

#### DMChannel#fetch(Pinned)Message(s)

`dmChannel.fetchMessage(s)` has been transformed in the shape of a DataStore.  See the [TextChannel#fetch(Pinned)Message(s)](/additional-info/changes-inv-v12?id=channel) section for more information.

#### DMChannel#search

`dmChannel.search()` has been removed entirely, along with all other user account-only properties and methods.

#### DMChannel#send\*\*\*

Just like the `TextChannel#send***` methods, all the `.send***()` methods have been removed in favor of one general `.send()` method. Read through the [TextChannel#send\*\*\*](/additional-info/changes-in-v12?id=channelsend) section for more information.

### Emoji

`Emoji` now extends `Base` and represent either a `GuildEmoji` or `ReactionEmoji`, and some of the specific properties have moved to their respective object, instead of everything on the base `Emoji` object.

#### Emoji#\*\*\*RestrictedRole(s)

The helper methods to add and remove a role or roles from the roles allowed to use the emoji have been removed from `emoji.edit()` and are now set via `guildEmoji.edit()`.

```diff
- emoji.addRestrictedRole('123456789012345678');
- emoji.addRestrictedRoles(['123456789012345678', '098765432109876543']);
- emoji.removeRestrictedRole('1234567890123345678');
- emoji.removedRestrictedRoles(['123456789012345678', '098765432109876543']);
+ emoji.edit({ roles: ['123456789012345678', '098765432109876543'] })
```

#### Emoji#deletable

`emoji.deletable` has been moved to `guildEmoji.deletable`.

#### Emoji#fetchAuthor

`emoji.fetchAuthor()` has been moved to `guildEmoji.fetchAuthor()`.

#### Emoji#guild

`emoji.guild` has been moved to `guildEmoji.guild`.

#### Emoji#setName

`emoji.setName()` has been moved to `guildEmoji.setName()`.

### EvaluatedPermissions

`evaluatedPermissions` has been removed entirely, see the `Permissions` page.

### Game

The `Game` class has been removed in favor of the `Activity` class to be consistent with the API.

```diff
- user.presence.game
+ user.presence.activity
```

### GroupDMChannel

The `GroupDMChannel` class has been deprecated from the Discord API.  While it's still available through Gamebridge for now, that will also be removed in the future.  In addition, group DM's has always been unreliable and hacky to work with a bot.

### Guild

#### Guild#acknowledge

`guild.acknowledge()` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#allowDMs

`guild.allowDMs()` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#ban

`guild.ban()` has been moved to the `GuildMemberStore`.  In addition, the second parameter in `guild.members.ban()` has been changed. The `options` parameter no longer accepts a number, nor a string.

```diff
- guild.ban(user, 7);
+ guild.members.ban(user, { days: 7 });

- guild.ban(user, 'Too much trolling');
+ guild.members.ban(user, { reason: 'Too much trolling' });
```

#### Guild#Channels

`guild.channels` is now a DataStore instead of a Collection.

#### Guild#createChannel

`guild.createChannel()` has been transformed in the shape of a DataStore.  The second, third and fourth parameters in `guild.createChannel()` have been changed/removed, leaving it with a total of two parameters, the second one being an object with all of the options available in `ChannelData`.

```diff
- guild.createChannel('new-channel', 'text', permissionOverwriteArray, 'New channel added for fun!');
+ guild.channels.create('new-channel', 'text', { overwrites: permissionOverwriteArray, reason: 'New channel added for fun!' });
```

#### Guild#createEmoji

`guild.createEmoji()` has been transformed in the shape of a DataStore.  The third and fourth parameters in `guild.createEmoji()` have been changed/removed, leaving it with a total of three parameters. The `roles` and `reason` parameters from v11 have been merged into an object as the third parameter.

```diff
- guild.createEmoji('./path/to/file.png', 'NewEmoji', collectionOfRoles, 'New emoji added for fun!');
+ guild.emojis.create('./path/to/file.png', 'NewEmoji', { roles: collectionOfRoles, reason: 'New emoji added for fun!' });
```

#### Guild#createRole

`guild.createRole()` has been transformed in the shape of a DataStore.  The first and second parameters in `guild.createRole()` have been changed/removed, leaving it with a total of one parameter. The `data` and `reason` parameters from v11 have been moved into an object as the first parameter.

```diff
- guild.createRole(roleData, 'New staff role!');
+ guild.roles.create({ data: roleData, reason: 'New staff role!' });
```

#### Guild#deleteEmoji

`Guild.deleteEmoji()` has been removed and transformed in the shape of a DataStore. Note the possible use of `resolve()` as a broader alternative to `get()`.

```diff
- guild.deleteEmoji('123456789012345678');
+ guild.emojis.resolve('123456789012345678').delete();
```

#### Guild#defaultChannel

Unfortunately, "default" channels don't exist in Discord anymore, and as such, the `guild.defaultChannel` property has been removed with no alternative.

**Q:** "I previously had a welcome message system (or something similar) set up using that property. What can I do now?"

**A:** There are a few ways to tackle this. Using the example of a welcome message system, you can:

1. Set up a database table to store the channel ID in a column when someone uses a `!welcome-channel #channel-name` command, for example. Then inside the `guildMemberAdd` event, use `client.channels.get('id')` and send a message to that channel. This is the most reliable method and gives server staff freedom to rename the channel as they please.
2. Make a new command that creates a `welcome-messages` channel, use `guild.channels.find('name', 'welcome-messages')`, and send a message to that channel. This method will work fine in most cases, but will break if someone on that server decides to rename the channel. This may also give you unexpected results, due to Discord allowing multiple channels to have the same name.

<p class="tip">Not sure how to set up a database? Check out [this page](/sequelize/)!</p>

#### Guild#emojis

`guild.emojis` has been transformed in the shape of a DataStore.

#### Guild#fetchBans

`guild.fetchBans()` will return a `Collection` of objects in v12, whereas v11 would return a `Collection` of `User` objects.

```diff
- guild.fetchBans().then(bans => console.log(`${bans.first().tag} was banned`));
+ guild.fetchBans().then(bans => console.log(`${bans.first().user.tag} was banned because "${bans.first().reason}"`));
```

#### Guild#fetchMember(s)

`guild.fetchMember()` and `guild.fetchMembers()` were both removed and transformed in the shape of DataStores. In addition, `guild.members.fetch()` will return a `Collection` of `GuildMember` objects in v12, whereas v11 would return a `Guild` object.

```diff
- guild.fetchMember('123456789012345678');
+ guild.members.fetch('123456789012345678');
```

```diff
- guild.fetchMembers();
+ guild.members.fetch();
```

#### Guild#iconURL

`guild.iconURL` is now a method, as opposed to a property. It also allows you to determine the file format and size to return.

```diff
- guild.iconURL;
+ guild.iconURL();
+ guild.iconURL({ format: 'png', size: 1024 });
```

#### Guild#messageNotifications

`guild.messageNotifications` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#mobilePush

`guild.mobilePush` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#muted

`guild.muted` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#position

`guild.position` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#presences

`guild.presences` is now a DataStore instead of a Collection.

#### Guild#pruneMembers

`guild.pruneMembers()` has been transformed in the shape of a DataStore.  In addition, the first, second, and third parameters in the method have been changed or removed, leaving it with a total of one parameter. The `days`, `dry`, and `reason` parameters from v11 have been merged into an object as the first parameter.

```diff
- guild.pruneMembers(7, true, 'Scheduled pruning');
+ guild.members.prune({ days: 7, dry: true, reason: 'Scheduled pruning' });
```

#### Guild#roles

`guild.roles` is now a DataStore instead of a Collection.

#### Guild#search

`guild.search()` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#setChannelPosition

`guild.setChannelPosition()` has been removed entirely. As an alternative, you can use `channel.setPosition()`, or `guild.setChannelPositions()`, which accepts accepts the same form of data as `guild.setChannelPosition` but inside an array.

```diff
- guild.setChannelPosition({ channel: '123456789012345678', position: 1 });
+ guild.setChannelPositions([{ channel: '123456789012345678', position: 1 }]);
+ channel.setPosition(1);
```

#### Guild#setPosition

`guild.setPosition()` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#setRolePosition

`guild.setRolePosition()` has been removed entirely as an extraneous helper method. As an alternative, you can use `role.setPosition()`.

```diff
- guild.setRolePosition({ role: '123456789012345678', position: 1 });
+ role.setPosition(1);
```

#### Guild#splashURL

`guild.splashURL` is now a method, as opposed to a property. It also allows you to determine the file format and size to return.

```diff
- guild.splashURL;
+ guild.splashURL();
+ guild.splashURL({ format: 'png', size: 1024 });
```

#### Guild#suppressEveryone

`guild.suppressEveryone` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#sync

`guild.sync()` has been removed entirely, along with all other user account-only properties and methods.

#### Guild#unban

`guild.unban()` has been transformed in the shape of a DataStore and is now a method on `GuildMemberStore`.  In addition, it also now optionally accepts a string as a second parameter for `reason`.

```diff
- guild.unban('123456789012345678');
+ guild.members.unban('123456789012345678', 'Ban appealed.');
```

### GuildChannel

The properties of a channel relating to its position have been renamed.  `guildChannel.calculatedPosition` is now `guildChannel.position`.  `guildChannel.position` is now more clearly named `guildChannel.rawPosition` to denote that it's directly from the API without any sorting.

```diff
- channel.calculatedPosition;
+ channel.position;

- channel.position;
+ channel.rawPosition;
```

#### GuildChannel#clone

The first, second, third, and fourth parameters in `channel.clone()` have been changed/removed, leaving it with a total of one parameter. The `name`, `withPermissions`, `withTopic`, and `reason` parameters from v11 have been merged into an object as the first parameter.  Several other parameters have also been added to the options object.

#### GuildChannel#createInvite

The second parameter in `channel.createInvite()` has been removed, leaving it with a total of one parameter. The `reason` parameter from v11 have been merged into an object as the first parameter.

```diff
- channel.createInvite({ temporary: true }, 'Just testing');
+ channel.createInvite({ temporary: true, reason: 'Just testing' });
```

#### GuildChannel#messageNotifications

`guildChannel.messageNotifications` has been removed entirely, along with all other user account-only properties and methods.

#### GuildChannel#muted

`guildChannel.muted` has been removed entirely, along with all other user account-only properties and methods.

#### GuildChannel#\*\*\*Permissions

=======
#### GuildChannel#members

`guildChannel.members` has been removed from `guildChannel.members` and added to `textChannel.members` and `voiceChannel.members`.

#### GuildChannel#messageNotifications

`guildChannel.messageNotifications` has been removed entirely.

#### GuildChannel#muted

`guildChannel.muted` has been removed entirely.

#### GuildChannel#\*\*\*Permissions

`guildChannel.memberPermissions` and `guildChannel.rolePermissions` are now private.

#### GuildChannel#replacePermissionOverwrites

`guildChannel.replacePermissionOverwrites` has been removed entirely.

#### GuildChannel#setPosition

The second parameter in `channel.setPosition()` has been changed. The `relative` parameter from v11 has been merged into an object.

```diff
- channel.setPosition(10, true);
+ channel.setPosition(10, { relative: true, reason: 'Basic organization' });
```

### GuildMember

#### GuildMember\*\*\*Role(s)

All of the methods to modify a member's roles have been moved to the `GuildMemberRoleStore`.

```diff
- guildMember.addRole('123456789012345678');
- guildMember.addRoles(['123456789012345678', '098765432109876543']);
+ guildMember.roles.add('123456789012345678');
+ guildMember.roles.add(['123456789012345678', '098765432109876543']);

- guildMember.removeRole('123456789012345678');
- guildMember.removeRoles(['123456789012345678', '098765432109876543']);
+ guildMember.roles.remove('123456789012345678');
+ guildMember.roles.remove(['123456789012345678', '098765432109876543']);

- guildMember.setRoles(['123456789012345678', '098765432109876543']);
+ guildMember.roles.set(['123456789012345678', '098765432109876543']);
```

#### GuildMember#ban

`guildMember.ban()` has been transformed in the shape of a DataStore and is now a method on `GuildMemberStore`. The second parameter has been changed from a string or an object to only accept an object.  The `reason` and `days` parameters are keys in the `options` object.

```diff
- member.ban(user, 7);
+ guild.members.ban(user, { days: 7 });

- member.ban(user, 'Too much trolling');
+ guild.members.ban(user, { reason: 'Too much trolling' });
```

#### GuildMember#\*\*\*Role

`guildMember.colorRole`, `guildMember.highestRole` and `guildMember.hoistRole` have all been moved to the `GuildMemberRoleStore` DataStore.

```diff
- guildMember.colorRole;
+ guildMember.roles.color;

- guildMember.highestRole;
+ guildMember.roles.highest;

- guildMember.hoistRole;
+ guildMember.roles.hoist;
```

#### GuildMember#\*\*\*deaf

`guildMember.deaf`, `guildMember.selfDeaf` and `guildMember.serverDeaf` have all been moved to the `VoiceState` class.

```diff
- guildMember.deaf;
+ guildMember.voice.deaf;

- guildMember.selfDeaf;
+ guildMember.voice.selfDeaf;

- guildMember.serverDeaf;
+ guildMember.voice.serverDeaf;
```

#### GuildMember#hasPermission

The `explicit` parameter has been removed entirely.  The `checkAdmin` and `checkOwner` parameters have been changed into a single `options` object with those values as keys.

```diff
- guildMember.hasPermission('MANAGE_MESSAGES', true, false, false);
+ guildMember.hasPermission('MANAGE_MESSAGES', { checkAdmin: false, checkOwner: false });
```

#### GuildMember#hasPermissions

`guildMember.hasPermissions()` has been removed in favor of `guildMember.hasPermission()`.

```diff
- guildMember.hasPermissions(['MANAGE_MESSAGES', 'MANAGE_ROLES']);
+ guildMember.hasPermission(['MANAGE_MESSAGES', 'MANAGE_ROLES']);
```

#### GuildMember#lastMessage

The `guildMember.lastMessage` property is now read-only.

#### GuildMember#missingPermissions

`guildMember.missingPermissions` has been removed entirely.

#### GuildMember#\*\*\*mute

`guildMember.mute`, `guildMember.selfMute` and `guildMember.serverMute` have all been moved to the `VoiceState` class.

```diff
- guildMember.mute;
+ guildMember.voice.mute;

- guildMember.selfMute;
+ guildMember.voice.selfMute;

- guildMember.serverMute;
+ guildMember.voice.serverMute;
```

#### GuildMember#roles

`guildMember.roles` is now a DataStore instead of a Collection.

#### GuildMember#send\*\*\*

Just like the `textChannel#send***` methods, all the `.send***()` methods have been removed in favor of one general `.send()` method. Read through the [textChannel#send\*\*\*](/additional-info/changes-in-v12?id=channelsend) section for more information.

#### GuildMember#speaking

`guildMember.speaking` has been moved to the `VoiceState` class.

```diff
- guildMember.speaking;
+ guildMember.voice.speaking;
```

#### GuildMember#voice\*\*\*

`guildMember.voiceChannel`, `guildMember.voiceChannelID` and `guildMember.voiceSessionID` have all been moved to the `VoiceState` class, which is read-only.

```diff
- guildMember.voiceChannel;
+ guildMember.voice.channel;

- guildMember.voiceChannelID;
+ guildMember.voice.channelID;

- guildMember.voiceSessionID;
+ guildMember.voice.sessionID;
```

### Invite

#### Invite#\*\*\*ChannelCount

`invite.textChannelCount` and `invite.voiceChannelCount` have both been removed entirely.

### Message

#### Message#acknowledge

`message.acknowledge()` has been removed entirely, along with all other user account-only properties and methods.

#### Message#clearReactions

`message.clearReactions()` has been transformed in the shape of a DataStore.

```diff
- message.clearReactions();
+ message.reactions.clear();
```

#### Message#delete

The first parameter in `message.delete()` has been changed. The `timeout` parameter from v11 have been merged into an object as the first parameter.  In addition, there is now another optional key in the object, `reason`.

```diff
- message.delete(5000);
+ message.delete({ timeout: 5000, reason: 'It had to be done.' });
```

#### Message#editCode

In the same sense that the `channel.sendCode()` method was removed, `message.editCode()` has also been removed entirely.

```diff
- message.editCode('js', 'const version = 11;');
+ message.edit('const version = 12;', { code: 'js' });
```

#### Message#hit

`message.hit` has been removed entirely, as it was used for user-account only searches.

#### Message#is(Member)Mentioned

`message.isMentioned()` and `message.isMemberMentioned()` have been removed in favor of `message.mentions.has()`.

```diff
- message.isMentioned('123456789012345678');
- message.isMemberMentioned('123456789012345678');
+ message.mentions.has('123456789012345678');
```

#### Message#member

`message.member` is now read-only.

### MessageAttachment

The `MessageAttachment` class constructor parameters have changed to reflect that `Attachment` has been removed and rolled into `MessageAttachment`.

#### MessageAttachment#client

`attachment.client` has been removed entirely so an attachment can be constructed without needing the full client.

#### MessageAttachment#filename

`attachment.filename` has been renamed to `attachment.name`.

#### MessageAttachment#filesize

`attachment.filesize` has been renamed to `attachment.size`.

### MessageCollector

See the `Collector` section for most of the changes to `MessageCollector`, such as the new `dispose` method and event.  Changes to the `MessageCollector` constructor in particular are as follows:

#### MessageCollector#channel

A `GroupDMChannel` is no longer able to be used for a collector, only `DMChannel` and `TextChannel`.

#### MessageCollector#message

The `messageCollector.message` event has been removed in favor of the generic `collector.on` event.

### MessageCollectorOptions

#### MessageCollectorOptions#max(Matches)

The `max` and `maxMatches` properties of the `MessageCollector` class have been renamed and repurposed.

```diff
- `max`: The The maximum amount of messages to process.
+ `maxProcessed`: The maximum amount of messages to process.

- `maxMatches`: The maximum amount of messages to collect.
+ `max`: The maximum amount of messages to collect.
```

### MessageEmbed

`MessageEmbed` now encompasses both the received embeds in a message and the constructor - the `RichEmbed` constructor was removed in favor of `MessageEmbed`.

#### MessageEmbed#attachFiles

`RichEmbed.attachFile()` is the only method that did not make the transition from v11 to v12.  The `MessageEmbed.attachFiles()` works for one or more files.

#### MessageEmbed#client

`messageEmbed.client` has been removed entirely so a new embed can be constructed without needing the full client.

#### MessageEmbed#message

`messageEmbed.message` has been removed entirely so a new embed can be constructed without needing the full client.

### MessageReaction

#### MessageReaction#fetchUsers

`messageReaction.fetchUsers()` has been transformed in the shape of a DataStore.  In addition, the first parameter has been removed in favor of an object.

```diff
- reaction.fetchUsers(50);
+ reaction.users.fetch({ limit: 50 });
```

#### MessageReaction#remove

`messageReaction.remove()` has been transformed in the shape of a DataStore.

```diff
- reaction.remove();
+ reaction.users.remove();
```

### OAuth2Application

The `OAuth2Application` class has been renamed to `ClientApplication`.

#### OAuth2Application#bot

`application.bot` has been removed entirely.

#### OAuth2Application#flags

`application.flags` has been removed entirely.

#### OAuth2Application#iconURL

`application.iconURL` is now a method, as opposed to a property. It also allows you to determine the file format and size to return.

```diff
- user.iconURL;
+ user.iconURL();
+ user.iconURL({ format: 'png', size: 1024 });
```

#### OAuth2Application#redirectURLs

`application.redirectURLs` has been removed entirely.

#### OAuth2Application#reset

`application.reset()` has been removed entirely, as it was an endpoint for user accounts.

#### OAuth2Application#rpcApplicationState

`application.rpcApplicationState` has been removed entirely.

#### OAuth2Application#secret

`application.secret` has been removed entirely.

### PartialGuild(Channel)

The `PartialGuild` and `PartialGuildChannel` classes for use with invites have been removed entirely.

### PermissionOverwrites

#### PermissionOverwrites#allow

`permissionOverwrites.allow` has been removed, use `permissionOverwrites.allowed` instead for a Permissions object of allowed permissions for the user or role.

#### PermissionOverwrites#deny

`permissionOverwrites.deny` has been removed, use `permissionOverwrites.denied` instead for a Permissions object of denied permissions for the user or role.

### Permissions

#### Permissions#_member

`permissions._member` has been removed entirely.

#### Permissions#flags

The following permission flags have been renamed:

* `READ_MESSAGES` -> `VIEW_CHANNEL`
* `EXTERNAL_EMOJIS` -> `USE_EXTERNAL_EMOJIS`
* `MANAGE_ROLES_OR_PERMISSIONS` -> `MANAGE_ROLES`

#### Permission#hasPermission(s)

`permissions.hasPermission()` and `permissions.hasPermissions()` have been removed entirely in favor of `permissions.has()`.  This change reduces extraneous helper methods.

#### Permissions#missingPermissions

`permissions.missingPermissions()` has been renamed to `permissions.missing()` and also refactored. The second parameter in v11 was named `explicit`, described as "Whether to require the user to explicitly have the exact permissions", defaulting to `false`. However, the second parameter in v11 is named `checkAdmin`, described as "Whether to allow the administrator permission to override", defaulting to `true`.

```diff
- permissions.missingPermissions(['MANAGE_SERVER']);
+ permissions.missing(['MANAGE_SERVER']);
```

#### Permissions#raw

`permissions.raw` has been removed in favor of `permissions.bitfield`.

```diff
- permissions.raw;
+ permissions.bitfield;
```

#### Permissions#resolve

`permissions.resolve()` has been removed entirely.

### Presence

#### Presence#game

`presence.game` has been removed in favor of the `Activity` class.

```diff
- presence.game;
+ presence.activity;
```

### Receiver
`receiver.createOpusStream(user)` and `receiver.createPCMStream(user)` have been removed in favor of `receiver.createStream()`.

```diff
- receiver.createOpusStream(message.author);
+ receiver.createStream(message.author, {mode: 'opus', end: 'silence'});
```

### RichEmbed

The `RichEmbed` class has been removed in favor of the `MessageEmbed` class.

#### RichEmbed#attachFile

`RichEmbed.attachFile()` has been removed in favor of `MessageEmbed.attachFiles()`.

```diff
- new RichEmbed().attachFile('attachment://file-namme.png');
+ new MessageEmbed().attachFiles(['attachment://file-namme.png']);

- new RichEmbed().attachFile({ attachment: './file-name.png' });
+ new MessageEmbed().attachFiles([{ attachment: './file-name.png' }]);

- new RichEmbed().attachFile(new Attachment('./file-name.png'));
+ new MessageEmbed().attachFiles([new MessageAttachment('./file-name.png')]);
```

### RichPresenceAssets

#### RichPresenceAssets#\*\*\*ImageURL

Both properties relating to the rich presence's image URL have been changed to be a method, as opposed to a property. It also allows you to determine the file format and size to return.

```diff
- asset.smallImageURL;
- asset.largeImageURL;
+ asset.smallImageURL();
+ asset.largeImageURL({ format: 'png', size: 1024 });
```

### Role

The properties of a role relating to its position have been renamed.  `role.calculatedPosition` is now `role.position`.  `role.position` is now more clearly named `role.rawPosition` to denote that it's directly from the API without any sorting.

```diff
- role.calculatedPosition;
+ role.position;

- role.position;
+ role.rawPosition;
```

#### Role#hasPermission(s)

`role.hasPermission()` and `role.hasPermissions()` have been removed in favor of `permissions.has()`.

```diff
- role.hasPermission('MANAGE_MESSAGES');
+ role.permissions.has('MANAGE_MESSAGES');
```

```diff
- role.hasPermissions(['MANAGE_MESSAGES', 'MANAGE_SERVER']);
+ role.permissions.has(['MANAGE_MESSAGES', 'MANAGE_SERVER']);
```

#### Role#serialize

`role.serialize()` has been removed as an extraneous helper method.

```diff
- role.serialize();
+ role.permissions.serialize();
```

#### Role#setPosition

The optional, second parameter of the `role.setPosition()` method has been changed to an object; its keys are `relative` (a boolean) and `reason` (a string).

```diff
- role.setPosition(3, true);
+ role.setPosition(3, { relative: true, reason: 'Needed to be higher.' });
```

### TextChannel

#### TextChannel#createCollector

`channel.createCollector()` has been removed in favor of `channel.createMessageCollector()`. In addition, the `max` and `maxMatches` properties were renamed and repurposed. You can read more about that [here](/additional-info/changes-in-v12?id=messagecollectormaxmatches).

```diff
- channel.createCollector(filterFunction, { maxMatches: 2, max: 10, time: 15000 });
+ channel.createMessageCollector(filterFunction, { max: 2, maxProcessed: 10, time: 15000 });
```

#### TextChannel#send\*\*\*

All the `.send***()` methods have been removed in favor of one general `.send()` method.

```diff
- channel.sendMessage('Hey!');
+ channel.send('Hey!');
```

```diff
- channel.sendEmbed(embedVariable);
+ channel.send(embedVariable);
+ channel.send({ embed: embedVariable });
```

<p class="warning">`channel.send(embedVariable)` will only work if that variable is an instance of the `MessageEmbed` class; object literals won't give you the expected result unless your embed data is inside an `embed` key.</p>

```diff
- channel.sendCode('js', 'const version = 11;');
+ channel.send('const version = 12;', { code: 'js' });
```

```diff
- channel.sendFile('./file.png');
- channel.sendFiles(['./file-one.png', './file-two.png']);
+ channel.send({
  files: [{
    attachment: 'entire/path/to/file.jpg',
    name: 'file.jpg',
  }]
+ channel.send({
  files: ['https://cdn.discordapp.com/icons/222078108977594368/6e1019b3179d71046e463a75915e7244.png?size=2048']
});
```

```diff
- channel.sendFiles(['./file-one.png', './file-two.png']);
+ channel.send({ files: [{ attachment: './file-one.png' }, { attachment: './file-two.png' }] });
+ channel.send({ files: [new MessageAttachment('./file-one.png'), new MessageAttachment('./file-two.png')] });
```

#### TextChannel#fetch(Pinned)Message(s)

`channel.fetchMessage()`, `channel.fetchMessages()`, and `channel.fetchPinnedMessages()` were all removed and transformed in the shape of DataStores.

```diff
- channel.fetchMessage('123456789012345678');
+ channel.messages.fetch('123456789012345678');
```

```diff
- channel.fetchMessages({ limit: 100 });
+ channel.messages.fetch({ limit: 100 });
```

```diff
- channel.fetchPinnedMessages();
+ channel.messages.fetchPinned();
```

### User

#### User#avatarURL

`user.avatarURL` is now a method, as opposed to a property. It also allows you to determine the file format and size to return.

```diff
- user.avatarURL;
+ user.avatarURL();
+ user.avatarURL({ format: 'png', size: 1024 });
```

#### User#displayAvatarURL

`user.displayAvatarURL` is now a method, as opposed to a property. It also allows you to determine the file format and size to return.

```diff
- user.displayAvatarURL;
+ user.displayAvatarURL();
+ user.displayAvatarURL({ format: 'png', size: 1024 });
```
### voiceConnection

#### VoiceConnection#createReceiver
`voiceconnection.createReceiver()` has been removed, the receiver can now simply be accessed from `voiceConnection.receiver`


### Webhook

#### Webhook#send\*\*\*

Just like the `TextChannel#send***` methods, all the `.send***()` methods have been removed in favor of one general `.send()` method. Read through the [TextChannel#send\*\*\*](/additional-info/changes-in-v12?id=channelsend) section for more information.

---

## Additions

<p class="warning">Remember to add examples for the additions.</p>

### Activity

### ActivityFlags

### APIMessage

### Base

### BaseClient

### BitField

### BroadcastDispatcher

### CategoryChannel

#### CategoryChannel#members

Similar to `textChannel#members` and `voiceChannel#members`, `categoryChannel#members` returns a `Collection` of `GuildMembers` who can see the category, mapped by their ID.

### Channel

#### Channel#toString

`channel.toString()` was moved from `GuildChannel` to `Channel`.

#### Channel#type

`channel.type` now may also return `unknown`.

### Client

#### Client#clearImmediate

#### Client#guildIntegrationsUpdate

#### Client#invalidated

#### Client#setImmediate

#### Client#webhookUpdate

### ClientApplication

This is a not a new class; it was formerly called `OAuth2Application` in v11.  Changes and deletions to methods and properties are covered above (link needed).  Additions are as follow:

#### ClientApplication#cover(Image)

`ClientApplication.cover` and its associated method `ClientApplication.coverImage()` return the URL to the application's cover image, with optional modifiers if applied in the method.

```js
ClientApplication.coverImage({ width: 1024, height: 1024 });
```

#### ClientApplication#fetchAssets

`ClientApplication.fetchAssests()` returns a promise that resolves into an array of `ClientAsset` objects, each of which contains `id`, `name` and `type` keys.

### ClientOptions

#### ClientOptions#partials

`clientOptions.partials` has been added to allow for partial structures - see the `Partials` section of the guide for more details.

#### ClientOptions#retry

`clientOptions.retry` has been added to allow a maximum amount of reconnect attempts on 5XX errors.

#### ClientOptions#presence

`clientOptions.presence` has been added to specify presence data to set upon login.

### ClientVoiceManager

### Collector

#### Collector#(handle)Dispose

`collector.handleDispose` and `collector.dispose()` have been added to remove an element from the collection.

### CollectorOptions

#### CollectorOptions#dispose

`collectorOptions.dispose` has been added to allow for deleted data to be removed from the collection.

### DataStore

The DataStore class was added in order to store various data types. Uses include
- RoleStore
- UserStore
- GuildStore
- ChannelStore
- MessageStore
- PresenceStore
- ReactionStore
- GuildEmojiStore
- GuildMemberStore
- GuildChannelStore
- ReactionUserStore
- GuildEmojiRoleStore
- GuildMemberRoleStore

### DiscordAPIError

#### DiscordAPIError#httpStatus

The `DiscordAPIError#httpStatus` has been added with the 4xx status code that the error returns.  See [the MDN docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#Client_error_responses) for more details.

### DMChannel

#### DMChannel#lastMessage

The message object of the last message in the channel, if one was sent.  It is a read-only property.

#### DMChannel#lastPin\*\*\*

Two properties have been added, `dmChannel#lastPinAt` (read-only) and `dmChannel#lastPinStamp`, which returns the Date and timestamp, respectively, of the last pinned message if one is present.

### Guild

#### Guild#createIntegration

`guild.createIntegration()` has been added.

#### Guild#fetchEmbed

`guild.fetchEmbed` has been added.

#### Guild#fetchIntegrations

`guild.fetchIntegrations()` has been added.

#### Guild#fetchVanityCode

`guild.fetchVanityCode()` has been added. 

#### Guild#setEmbed

`guild.setEmbed()` has been added.

#### Guild#shard(ID)

`guild.shard` (read-only) and `guild.shardID` have been added, representing the information of the shard the guild is on.

### GuildAuditLogs

#### GuildAuditLogs#Actions

`auditLogs.Actions()` has been added (static method).

#### GuildAuditLogs#Targets

`auditLogs.Targets()` has been added (static method).

### GuildChannel

#### GuildChannel#permissionsLocked

`guildChannel.permissionsLocked` is a boolean value representing if the `permissionOverwrites` of the channel match its parent's `permissionOverwrites`.

#### GuildChannel#updateOverwrites

Creates or update an existing overwrite for a user or role.  The first parameter is a `PermissionOverwriteOption` object; the second, optional parameter is `reason`, a string.

```js
channel.updateOverwrite(message.author, {
	SEND_MESSAGES: false,
});
```

#### GuildChannel#viewable

`guildChannel.viewable` is a boolean value representing whether the channel is visible to the client user.

### HTTPError

### Integration

### Message

#### Message#activity

`message.activity` has been added.

#### Message#application

`message.application` has been added.

#### Message.url

`message.url` has been added in order to provide a URL to jump to the message.

### MessageAttachment

#### MessageAttachment#setAttachment

`attachment.setAttachment()` has been added.

#### MessageAttachment#setFile

`attachment.setFile()` has been added.

#### MessageAttachment#setName

`attachment.setName()` has been added.

### MessageEmbed

#### MessageEmbed#files

`MessageEmbed.files` has been added as an array of files in the `MessageEmbed`.

#### MessageEmbed#length

`MessageEmbed.length` has been added.  It returns a `number` equal to all of the fields, title, description, and footer.

### MessageMentions

#### MessageMentions#has

`mentions.has()` has been added, replacing `message.isMentioned()` and `message.isMemberMentioned()`.  It has two paramets: the first is `data` representing a `User`, `GuildMember`, `Role`, or `GuildChannel` and an optional `options` object.

```diff
- message.isMentioned('123456789012345678');
- message.isMemberMentioned('123456789012345678');
+ message.mentions.has('123456789012345678');
+ message.mentions.has('123456789012345678', { ignoreDirect: true, ignoreRoles: true, ignoreEveryone: true });
```

###

### Permission

#### Permissions#flags
`PRIORITY_SPEAKER` has been added.

### PermissionOverwrites

#### PermissionOverwrites#allowed

`permissionOverwrites.allowed` has been added.

#### PermissionOverwrites#denied

`permissionOverwrites.denied` has been added.

### Presence

#### Presence#clientStatus

The new `presence.clientStatus` property returns an object with three keys: `web`, `mobile` and `desktop`; their values are a `PresenceStatus` string.  This property allows you to check which client the member or user is using to access Discord.

#### Presence#guild

`presence.guild` has been added as a helper property to represent the `Guild` the presence belongs to, if applicable.

#### Presence#member

`presence.member` is a read-only property to represent the `GuildMember` the presence belongs to, if applicable.

#### Presence#user(ID)

`presence.user` (read-only) and `presence.userID` are properties to represent a `User` and its ID that the presence belongs to.  The former can be null if the `User` is not cached.

### ReactionCollector

#### ReactionCollector#empty

`reactionCollector.empty()` has been added as a method to remove all collected reactions from the collector.

#### ReactionCollector#key

#### ReactionCollector#remove

The new `remove` event emits when a collected reaction is un-reacted, if the `dispose` option is set to `true`

### TextChannel#lastPinTimestamp
`TextChannel.lastPinTimestamp` was added.

### TextChannel#lastPinAt
`TextChannel.lastPinAt` was added.

### User

#### User#local
`user.local` has been added

### Util#Constants

#### Constant.Colors
`WHITE` was added as a value.