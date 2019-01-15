# Complex Class Relationships

## Learning Goals

- Recognize how there are many ways to design relationships between classes
- Recognize how high cohesion, weak coupling and the application of the single responsibility principle tend to encourage one-way relationships

## Introduction

In the previous lessons, we've explored building one-way relationships between
two classes. For instance, a `Book` instance can have an `Author` instance
stored as a property, making it dependent on `Author`. Alternatively, we could
establish a relationship where an `Author` instance has many `Book`s, making it
dependent on `Book`.

As we add more classes, however, things become more complicated, and structure
_matters_.

In this lesson, we're going to look at a few, more complex examples of how
classes can have relationships and touch upon some of the issues to watch out
for when designing classes that work together.

## Deciding How to Structure Complex Relationships

Imagine we're building an app to organize a music collection and we want to
represent a relationship between an artist, their albums, and their songs. We can
imagine in real life that an artist has many albums and an album has many songs,
so maybe when representing in Object Orientation, we could argue an `Artist`
instance should have access to its albums, and an album should have access to
its songs. That is, the _artist_ and _album_ should maintain the dependencies:

```js
// the Song class only serves up its own info
class Song {
	constructor(title) {
		this._title = title;
	}

	get title() {
		return this._title;
	}
}

// the Album class serves up its own info and contains a collection of Song instances
class Album {
	constructor(title, songs = []) {
		this._title = title;
		this._songs = songs;
	}

	get songs() {
		return this._songs;
	}

	set songs(songs) {
		this._songs = songs;
	}
}

// the Artist class serves up its own info and contains a collection of Album instances
class Artist {
	constructor(name, albums = []) {
		this._name = name;
		this._albums = albums;
	}

	get albums() {
		return this._albums;
	}

	set albums(albums) {
		this._albums = albums;
	}
}

let song = new Song('King of Cool');
let album = new Album('is that my engine or the song?', [song]);

let artist = new Artist('Cool Timmy', [album]);

artist;
// => Artist {
//     _name: 'Cool Timmy',
//     _albums:
//      [ Album { _title: 'is that my engine or the song?', _songs: [Array] } ] }
```

Since `Artist` is now dependent on `Album` and `Album` is dependent on `Song`,
we could visually represent this relationship as follows:

<img src="https://curriculum-content.s3.amazonaws.com/fewpjs/fewpjs-class-relationships/artist_album_song.png" width: 50% />

The arrows represent the dependencies between `Artist`, `Album` and `Song`. In
this configuration, from an `Artist` instance, we can access any associated
albums. Through those albums, we can access any associated songs. A
song, however, does not know the album it belongs to, and an album does not know
its artist.

What about an alternative set up? It could be argued that a collection of music is _actually_ made up of albums primarily - albums have both an artist and songs. Maybe the `Album` class should maintain the relationships? That might look like the following:

```js
// now, the Artist and Song classes only serves up their own info
class Artist {
	constructor(name) {
		this._name = name;
	}

	get name() {
		return this._name;
	}
}

class Song {
	constructor(title) {
		this._title = title;
	}

	get name() {
		return this._name;
	}
}

// the Album class contains a property for the artist it belongs to and the songs it has
class Album {
	constructor(title, artist, songs) {
		this._title = title;
		this._artist = artist;
		this._songs = songs;
	}

	get artist() {
		return this._artist;
	}

	get songs() {
		return this._songs;
	}
}

let artist = new Artist('Cool Timmy');

let theSteed = new Song('The Steed');
let kingOfCool = new Song('King of Cool');

let album = new Album('is that my engine or the song?', artist, [
	theSteed,
	kingOfCool
]);

album;
// => Album {
//     _title: 'is that my engine or the song?',
//     _artist: Artist { _name: 'Cool Timmy' },
//     _songs:
//      [ Song { _title: 'The Steed' },
//        Song { _title: 'King of Cool' } ] }
```

`Album` is now dependent on `Song` and `Artist`. `Artist` and `Song` instances
do not know about their `Album` instance.

<img src="https://curriculum-content.s3.amazonaws.com/fewpjs/fewpjs-class-relationships/album_artist_song.png" width: 50% />

## Considering Two-Way Dependencies

There are other options we could choose. We could argue that the three classes
should be _dependent upon each other_. Maybe a song should know its album and
artist, an artist should know their songs and albums, and an album should know
its artist and songs:

<img src="https://curriculum-content.s3.amazonaws.com/fewpjs/fewpjs-class-relationships/song_album_artist_two_way.png" width: 50% />

Its possible to make this work, but two-way dependencies have some caveats.
`Artist`, `Album` and `Song` would need to each keep track of each other. If an
`Album` instance was assigned to an `Artist` property, we would need to also make
sure that `Artist` instance is assigned to the `Album` instance.

Information about a relationship is being maintained from both sides, creating
multiple [sources of truth][truth]. When there is more than one source of
information, there is the potential for this information to get misaligned.
Somewhere in our code, for instance, we might forget to update one side of the
relationship, causing errors. To prevent this from happening, we have to add in
additional logic to ensure all sources of information are kept consistent. This
usually makes the code _more_ complicated than any value a two-way dependency
might provide.

Two-way dependencies strengthen coupling, and strong coupling tends to
make code less flexible and harder to update.

## Applying the Single Responsibility Principle to the Problem

There is still _another_ option to consider. What if we were to set up class
relationships where `Artist`, `Album` and `Song` are not dependent on each other
at all? Let's go back to the original design:

<img src="https://curriculum-content.s3.amazonaws.com/fewpjs/fewpjs-class-relationships/artist_album_song.png" width: 50% />

How might we change this so that `Artist` is not dependent on `Album`, and
`Album` is not dependent on `Song`?

Think about it this way - what are the responsibilities of `Artist`, `Album` and `Song` in this example?

- An `Artist` instance serves up its own info, `name`
- An `Artist` instance keeps track of associated `Album` instances
- An `Album` instance serves up its own info, `title`
- An `Album` instance keeps track of associated `Song` instances
- A `Song` instance serves up its own info, `title`

_Should_ an `Artist` instance need to keep track of its `Albums`? Should any of
these classes need keep track of any others?

Seems like this might violate the single responsibility principle!

As per SRP, `Artist`, `Album` and `Song` should have one responsibility each, and
it makes the most sense that this responsibility is just to serve up data about
themselves. This would make our classes highly cohesive.

It means, though, that we need to create a fourth class - a class that that
serves to join instances of an `Artist`, an `Album` and many `Songs`. At this
point, we'll have to move away from classes that represent things in the real
world.

Instead, we need a class that only serves to establish the relationship between
an artist, an album and its songs. This class acts as a sort of 'container' - an
object _comprised_ of an `Artist`, an `Album` and `Song` instances:

<img src="https://curriculum-content.s3.amazonaws.com/fewpjs/fewpjs-class-relationships/record_container.png" width: 50% />

Now, `Artist`, `Album`, and `Song` don't know about each other at all, but we're
still able to preserve their relationships. Our three initial classes have
become very simple:

```js
class Artist {
	constructor(name) {
		this._name = name;
	}

	get name() {
		return this._name;
	}
}

class Song {
	constructor(title) {
		this._title = title;
	}

	get title() {
		return this._title;
	}
}

class Album {
	constructor(title) {
		this._title = title;
	}

	get title() {
		return this._title;
	}
}

class RecordContainer {
	constructor(album, artist, songs) {
		this._album = album;
		this._artist = artist;
		this._songs = songs;
	}

	get album() {
		return this._album;
	}
	get artist() {
		return this._artist;
	}
	get songs() {
		return this._songs;
	}
}

let arianaGrande = new Artist('Ariana Grande');
let raindrops = new Song('Raindrops');
let blazed = new Song('Blazed');
let sweetener = new Album('Sweetener');
let record = new RecordContainer(sweetener, arianaGrande, [raindrops, blazed]);

record;
// => RecordContainer {
//   _album: Album { _title: 'Sweetener' },
//   _artist: Artist { _name: 'Ariana Grande' },
//   _songs: [ Song { _title: 'Raindrops' }, Song { _title: 'Blazed' } ] }
record.album.title;
// => 'Sweetener'
record.artist.name;
// => 'Ariana Grande'
record.songs;
// => [ Song { _title: 'Raindrops' }, Song { _title: 'Blazed' } ]
```

In this example, the only responsibility `RecordContainer` has is to establish
the relationships between an artist, an album and the songs on that album. We've
maintained the single responsibility principle! Although `RecordContainer` is
dependent on three classes, since `Artist`, `Album` and `Song` are not dependent
on anything, coupling is still weak overall.

Instead of having to write a lot of custom logic in our classes (as we might if
writing a two-way dependency mentioned earlier), _all_ four classes are generic
and easy to read.

`RecordContainer` serves as a sort of connector, forming a _unit_ comprised of
itself and the `Artist`, `Album` and `Song` classes. Since it maintains the
relationship, and therefore access to `Artist`, `Album`, and `Song` instance
data, it serves as an entry point for other classes that might utilize this
data.

## Conclusion

Object Orientation allows us represent real world relationships, which can make
it easier to understand and model in our heads. We _can_ create dependencies
that mirror real world relationships - an album _has_ an artist and songs.
Object Oriented design suggests something else, however. If we strive to
maintain high cohesion, weak coupling, and follow the single responsibility
principle, we begin to move away from classes _strictly_ representing real world
things. We may need to utilize additional classes to handle abstract concepts,
such as a relationship.

When an Object Oriented application becomes large and complex, however,
following good design principles will lead to easier to understand code.

## Resources

- [Single Source of Truth][truth]

[truth]: https://en.wikipedia.org/wiki/Single_source_of_truth
