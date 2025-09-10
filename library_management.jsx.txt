import React, { useEffect, useState } from "react";

// Library Management UI
// Features: Search, Add book, Remove book, persistent via localStorage, simple validation
// Styling: Tailwind CSS classes (with gradient enhancements)

export default function LibraryApp() {
  const [books, setBooks] = useState(() => {
    try {
      const raw = localStorage.getItem("library_books_v1");
      return raw ? JSON.parse(raw) : sampleBooks();
    } catch (e) {
      return sampleBooks();
    }
  });

  const [query, setQuery] = useState("");
  const [showForm, setShowForm] = useState(false);
  const [form, setForm] = useState({ title: "", author: "", year: "" });
  const [error, setError] = useState("");
  const [sortBy, setSortBy] = useState("title");

  useEffect(() => {
    localStorage.setItem("library_books_v1", JSON.stringify(books));
  }, [books]);

  function sampleBooks() {
    return [
      { id: id(), title: "The Alchemist", author: "Paulo Coelho", year: 1988 },
      { id: id(), title: "Clean Code", author: "Robert C. Martin", year: 2008 },
      { id: id(), title: "You Don\'t Know JS", author: "Kyle Simpson", year: 2015 },
    ];
  }

  function id() {
    return Math.random().toString(36).slice(2, 9);
  }

  function handleSearchChange(e) {
    setQuery(e.target.value);
  }

  function filteredBooks() {
    const q = query.trim().toLowerCase();
    const result = books.filter((b) => {
      if (!q) return true;
      return (
        b.title.toLowerCase().includes(q) ||
        (b.author && b.author.toLowerCase().includes(q)) ||
        (b.year && String(b.year).includes(q))
      );
    });

    return result.sort((a, b) => {
      if (sortBy === "title") return a.title.localeCompare(b.title);
      if (sortBy === "author") return a.author.localeCompare(b.author);
      if (sortBy === "year") return (a.year || 0) - (b.year || 0);
      return 0;
    });
  }

  function handleInput(e) {
    setForm((s) => ({ ...s, [e.target.name]: e.target.value }));
    setError("");
  }

  function handleAddBook(e) {
    e.preventDefault();
    const { title, author, year } = form;
    if (!title.trim()) return setError("Title is required");
    const newBook = { id: id(), title: title.trim(), author: author.trim(), year: year ? Number(year) : undefined };
    setBooks((s) => [newBook, ...s]);
    setForm({ title: "", author: "", year: "" });
    setShowForm(false);
  }

  function handleRemoveBook(bookId) {
    if (!confirm("Do you really want to remove this book?")) return;
    setBooks((s) => s.filter((b) => b.id !== bookId));
  }

  function handleClearAll() {
    if (!books.length) return;
    if (!confirm("Clear all books from the library?")) return;
    setBooks([]);
  }

  return (
    <div className="min-h-screen bg-gradient-to-br from-indigo-50 via-white to-pink-50 p-6 md:p-12 font-sans">
      <div className="max-w-4xl mx-auto bg-gradient-to-tr from-white via-indigo-50 to-purple-50 rounded-2xl shadow-lg p-6 md:p-10">
        <header className="flex flex-col md:flex-row md:items-center md:justify-between gap-4">
          <div>
            <h1 className="text-2xl md:text-3xl font-bold bg-gradient-to-r from-indigo-600 to-purple-600 bg-clip-text text-transparent">
              Library Management
            </h1>
            <p className="text-sm text-gray-500">Search, add, and remove books — lightweight and responsive.</p>
          </div>

          <div className="flex items-center gap-3">
            <div className="relative">
              <input
                aria-label="Search books"
                value={query}
                onChange={handleSearchChange}
                placeholder="Search by title, author or year..."
                className="w-64 md:w-80 rounded-lg border px-3 py-2 pr-10 text-sm focus:ring-2 focus:ring-indigo-300"
              />
              <button
                onClick={() => setQuery("")}
                className="absolute right-1 top-1/2 -translate-y-1/2 text-xs px-2 py-1 rounded bg-gradient-to-r from-gray-100 to-gray-200"
              >
                Clear
              </button>
            </div>

            <select
              value={sortBy}
              onChange={(e) => setSortBy(e.target.value)}
              className="rounded-lg border px-2 py-2 text-sm bg-gradient-to-r from-indigo-50 to-purple-50"
              aria-label="Sort books"
            >
              <option value="title">Sort: Title</option>
              <option value="author">Sort: Author</option>
              <option value="year">Sort: Year</option>
            </select>

            <button
              onClick={() => setShowForm(true)}
              className="rounded-md bg-gradient-to-r from-indigo-600 to-purple-600 text-white px-4 py-2 text-sm hover:from-indigo-700 hover:to-purple-700"
            >
              + Add Book
            </button>

            <button
              onClick={handleClearAll}
              className="rounded-md bg-gradient-to-r from-red-50 to-red-100 text-red-700 px-3 py-2 text-sm border border-red-200"
            >
              Clear All
            </button>
          </div>
        </header>

        <main className="mt-6">
          <BookList books={filteredBooks()} onRemove={handleRemoveBook} />
        </main>
      </div>

      {/* Add Book Modal/Form */}
      {showForm && (
        <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/40 p-4">
          <form onSubmit={handleAddBook} className="w-full max-w-md bg-gradient-to-br from-white via-indigo-50 to-purple-50 rounded-xl p-6 shadow-lg">
            <div className="flex items-center justify-between">
              <h2 className="text-lg font-medium bg-gradient-to-r from-indigo-600 to-purple-600 bg-clip-text text-transparent">Add a new book</h2>
              <button type="button" onClick={() => setShowForm(false)} className="text-gray-500">Close</button>
            </div>

            <div className="mt-4 flex flex-col gap-3">
              <label className="text-sm">Title *</label>
              <input name="title" value={form.title} onChange={handleInput} className="rounded border px-3 py-2" />

              <label className="text-sm">Author</label>
              <input name="author" value={form.author} onChange={handleInput} className="rounded border px-3 py-2" />

              <label className="text-sm">Year</label>
              <input name="year" value={form.year} onChange={handleInput} type="number" className="rounded border px-3 py-2" />

              {error && <div className="text-red-600 text-sm">{error}</div>}

              <div className="flex justify-end gap-3 mt-2">
                <button type="button" onClick={() => setShowForm(false)} className="px-3 py-2 rounded border">Cancel</button>
                <button type="submit" className="px-4 py-2 rounded bg-gradient-to-r from-indigo-600 to-purple-600 text-white">Add Book</button>
              </div>
            </div>
          </form>
        </div>
      )}
    </div>
  );
}

function BookList({ books, onRemove }) {
  if (!books.length) {
    return (
      <div className="p-8 text-center text-gray-500">
        No books found. Try adding some.
      </div>
    );
  }

  return (
    <ul className="mt-4 grid grid-cols-1 sm:grid-cols-2 gap-4">
      {books.map((b) => (
        <li key={b.id} className="flex items-start gap-4 p-4 rounded-lg border hover:shadow bg-gradient-to-r from-white to-indigo-50">
          <div className="w-12 h-16 rounded bg-gradient-to-br from-gray-100 to-gray-200 flex items-center justify-center text-xs font-semibold text-gray-600">
            {b.title.slice(0, 2).toUpperCase()}
          </div>

          <div className="flex-1">
            <div className="flex items-center justify-between gap-4">
              <div>
                <div className="font-semibold">{b.title}</div>
                <div className="text-sm text-gray-500">{b.author || "Unknown author"}</div>
              </div>

              <div className="text-sm text-gray-400">{b.year || "—"}</div>
            </div>

            <div className="mt-3 flex gap-2">
              <button onClick={() => onRemove(b.id)} className="px-3 py-1 rounded border text-sm text-red-600 bg-gradient-to-r from-red-50 to-red-100">Remove</button>
            </div>
          </div>
        </li>
      ))}
    </ul>
  );
}
