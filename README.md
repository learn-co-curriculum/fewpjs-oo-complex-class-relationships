# Complex Class Relationships

## Learning Goals

- Recognize how many `class`es working together can build a larger application
- Recognize how `class`es can be dependent on each other in a variety of
  configurations

## Introduction

In the previous lessons, we've explored building one-way relationships between
two classes. For instance, a `Book` instance can have an `Author` instance
stored as a property, making it dependent on `Author`. Or, alternatively, an
`Author` instance might have many `Book`s, making it dependent on `Book`.

When we add more classes, however, things become more complicated, and structure
_matters_.

In this lesson, we're going to look at a few, more complex examples of how
classes can have relationships and touch upon some of the issues to watch out
for.

## Two Way Relationships

In some applications, it is sometimes necessary for two classes to be _dependent
upon each other_. For example, lets say we have `Artist` and `Song` classes - it
might make sense to have a `Song` instance keep track of its `Artist` while at
the same time, have an `Artist` instance keep track of an array of `Song`s. For
example:

```js
// an Artist instance starts with an empty array that will contain Song instances
class Artist {
	constructor(name) {
		this.name = name;
		this._songs = [];
	}

	get songs() {
		return this._songs;
	}

	set songs(songs) {
		this._songs = songs;
	}

	get songTitles() {
		return this._songs.map(song => song.title);
	}
}

// Song takes in an instance of Artist when created
class Song {
	constructor(title, genre, artist, publishingDate) {
		this._title = title;
		this._genre = genre;
		this._artist = artist;
		this._publishingDate = publishingDate;

		// when a Song instance is created, it updates the 'artist.songs' pseudo-property
		artist.songs = [...artist.songs, this];
	}

	get title() {
		return this._title;
	}
}

let arianaGrande = new Artist('Ariana Grande');
new Song('thank u, next', 'Pop', arianaGrande, '2018');
new Song('Bang Bang', 'Pop', arianaGrande, '2014');

arianaGrande.songs;
// => [ Song {
//     _title: 'thank u, next',
//     _genre: 'Pop',
//     _artist: Artist { name: 'Ariana Grande', _songs: [Circular] },
//     _publishingDate: '2018' },
//     Song {
//     _title: 'Bang Bang',
//     _genre: 'Pop',
//     _artist: Artist { name: 'Ariana Grande', _songs: [Circular] },
//     _publishingDate: '2014' } ]
```

> Notice that each Song has an `_artist` property that also contains a `_songs`
> property! If you run the code above in Node, it recognizes that this is a
> circular reference

Here, we've got a two way dependency. The `Song` class is passed an instance of
`Artist` when created. Since the `Artist` instance carries around its getter and
setter methods, we can use them in the `Song` constructor.

With this set up, `artist.songs` serves as a way to maintain consistent data -
when a song is created, it is always added to the array of songs for the artist
that wrote it. In the code above, we didn't even need to store our `Song`
instances in variables when created, since they were immediately added to
`arianaGrande`.

Both `Artist` and `Song` rely on methods contained in the other, so we've
established a two way relationship. These types of relationships are also
refered to as _associations_. A `Song` _has_ an `Artist` and an `Artist` _has
many_ songs.

**Note:** We will get into Object Oriented design soon, but there is one caveat
about two-way relationships to remember now. In short, _more dependencies_
results in code that is less flexible and harder to change. Sticking to the
fewest dependencies necessary is typically preferred, and, depending on the
design of your application, _you probably don't need a two-way relationship_.

Two-way relationships break the principle of maintaining a [single source of
truth][truth]. We can get information from two sources, `Artist` and `Song`, but
it is possible for them to provide inconsistent information.

## Multi-Level Relationships

Most often, Object Oriented applications are comprised of many classes,
connected together. Returning to our `Book` and `Author` example, lets say we
want to create an application to represent an entire library's _catalog_, a
register of all books the library has available. We could include an additional
class, `Catalog`, that contains an array for containing `Book` instances along
with some methods to provide a user interface for organizing and locating books,
authors and genres.

```js
class Genre {
	constructor(name) {
		this._name = name;
	}
}

// for simplicity, Author just has one `_name` property
class Author {
	constructor(name) {
		this._name = name;
	}

	get name() {
		return this._name;
	}
}

// a Book takes in Genre and Author instances when created
class Book {
	constructor(title, author, genre, publishingDate) {
		this._title = title;
		this._author = author;
		this._genre = genre;
		this._publishingDate = publishingDate;
	}

	get title() {
		return this._title;
	}

	get author() {
		return this._author;
	}

	get genre() {
		return this._genre;
	}

	get publishingDate() {
		return this._publishingDate;
	}
}

// Catalog serves as a container for Book instances
class Catalog {
	constructor(libraryName) {
		this._libraryName = libraryName;
		this._books = [];
	}

	// an example method for finding a book based on the author's name
	// this method checks the name property belonging to the author property belonging to each book for a match
	findBooksByAuthorName(authorName) {
		return this.books.filter(book => book.author.name.includes(authorName));
	}

	get books() {
		return this._books;
	}

	set books(books) {
		this._books = books;
	}
}

// with the classes above, we can create Catalog, Genre and Author instances
let library = new Catalog('New York Public');
let scifi = new Genre('Sci-Fi');
let neal = new Author('Neal Stephenson');
let verner = new Author('Verner Vinge');

// then use our Genre and Author instances to create some Book instances
let snowCrash = new Book('Snow Crash', neal, scifi, '1992');
let thePeaceWar = new Book('The Peace War', verner, scifi, '1984');

// and add these books to the library catalog
library.books = [...library.books, snowCrash];
library.books = [...library.books, thePeaceWar];

// once books are added, we can test out the example method for finding by author
library.findBooksByAuthorName('Neal');
// => [ Book {
// _title: 'Snow Crash',
// _author: Author { _name: 'Neal Stephenson' },
// _genre: Genre { _name: 'Sci-Fi' },
// _publishingDate: '1992' } ]
```

In the code snippet above, we've got four related classes. Although a `Catalog`
instance _only_ contains an array of `Book` instances, because each `Book`
instance is related to an `Author` and a `Genre`, we can access that information
from within `Catalog`, as we see with `library.findBooksByAuthorName`. A library
catalog has many authors and genres _through_ books.

One important thing to note here - we _could have_ created an `Author` class
that kept track of each book an author has written, but we don't need to (and as
mentioned before, this breaks the single source of truth principle). The
relationships we've established will still allow us to write code that finds all
books belonging to an author.

But even better, there is no chance that data might be inconsistent. A `Book` is
the source of truth for its author and genre. The `Catalog` is the source of
truth for all available books.

## Conclusion

It is ultimately up to you to decide which classes are dependent on others, and
how many classes you will need. As you create Object Oriented applications,
however, always keep relationships in your mind. In general, fewer dependencies
minimizes complicated relationships. More classes can help divide
responsibilities, making your code easier to understand and maintain.

## Resources

- [Single Source of Truth][truth]

[truth]: https://en.wikipedia.org/wiki/Single_source_of_truth
