import React, { SEO master } from "react";
import { motion } from "framer-motion";
import { Plus, Pencil, Trash2, Home, FileText, Save, X, Calendar, Tag } from "lucide-react";

// --- Utility functions ---
const slugify = (str) =>
  str
    .toString()
    .toLowerCase()
    .trim()
    .replace(/\s+/g, "-")
    .replace(/[^a-z0-9-]/g, "")
    .replace(/-+/g, "-");

const LS_KEY = "book_to_book_posts_v1";

function loadPosts() {
  try {
    const raw = localStorage.getItem(LS_KEY);
    return raw ? JSON.parse(raw) : [];
  } catch (e) {
    console.error(e);
    return [];
  }
}

function savePosts(posts) {
  localStorage.setItem(LS_KEY, JSON.stringify(posts));
}

// --- Components ---
function Navbar({ route, setRoute }) {
  const link = (id, Icon, label) => (
    <button
      onClick={() => setRoute(id)}
      className={`flex items-center gap-2 px-4 py-2 rounded-2xl transition shadow-sm border ${
        route === id
          ? "bg-black text-white border-black"
          : "bg-white hover:bg-gray-50 border-gray-200"
      }`}
    >
      <Icon className="w-4 h-4" />
      <span className="text-sm font-medium">{label}</span>
    </button>
  );

  return (
    <div className="sticky top-0 z-20 bg-white/70 backdrop-blur supports-[backdrop-filter]:bg-white/60 border-b border-gray-100">
      <div className="max-w-5xl mx-auto px-4 py-3 flex items-center justify-between">
        <div className="flex items-center gap-3">
          <div className="w-9 h-9 rounded-2xl bg-black text-white grid place-items-center font-bold">B</div>
          <div className="font-semibold">Book to Book Side</div>
        </div>
        <div className="flex gap-2">
          {link("home", Home, "Home")}
          {link("blog", FileText, "Blog")}
        </div>
      </div>
    </div>
  );
}

function EmptyState({ onCreate }) {
  return (
    <div className="text-center py-20">
      <div className="mx-auto w-16 h-16 rounded-2xl bg-gray-100 grid place-items-center">
        <FileText className="w-7 h-7 text-gray-500" />
      </div>
      <h3 className="mt-6 text-lg font-semibold">No posts yet</h3>
      <p className="text-gray-500 mt-2">Click below to write your first blog post.</p>
      <button
        onClick={onCreate}
        className="mt-6 px-4 py-2 rounded-2xl bg-black text-white inline-flex items-center gap-2"
      >
        <Plus className="w-4 h-4" /> New Post
      </button>
    </div>
  );
}

function PostCard({ post, onEdit, onDelete, onOpen }) {
  return (
    <motion.div
      layout
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      className="p-5 rounded-2xl border border-gray-200 bg-white shadow-sm"
    >
      <div className="flex items-start justify-between gap-4">
        <div className="flex-1">
          <h3
            onClick={() => onOpen(post)}
            className="text-lg font-semibold cursor-pointer hover:underline"
          >
            {post.title}
          </h3>
          <div className="flex items-center gap-3 text-sm text-gray-500 mt-1">
            <span className="inline-flex items-center gap-1">
              <Calendar className="w-4 h-4" /> {new Date(post.date).toLocaleDateString()}
            </span>
            {post.tags?.length ? (
              <span className="inline-flex items-center gap-1">
                <Tag className="w-4 h-4" /> {post.tags.join(", ")}
              </span>
            ) : null}
          </div>
          <p className="text-gray-600 mt-3 line-clamp-2">
            {post.excerpt || post.content.slice(0, 140) + (post.content.length > 140 ? "…" : "")}
          </p>
        </div>
        <div className="flex items-center gap-2">
          <button
            onClick={() => onEdit(post)}
            className="px-3 py-2 rounded-xl border border-gray-200 hover:bg-gray-50 inline-flex items-center gap-2"
          >
            <Pencil className="w-4 h-4" /> Edit
          </button>
          <button
            onClick={() => onDelete(post)}
            className="px-3 py-2 rounded-xl border border-gray-200 hover:bg-gray-50 inline-flex items-center gap-2 text-red-600"
          >
            <Trash2 className="w-4 h-4" /> Delete
          </button>
        </div>
      </div>
    </motion.div>
  );
}

function PostForm({ initial, onCancel, onSave }) {
  const [title, setTitle] = useState(initial?.title || "");
  const [content, setContent] = useState(initial?.content || "");
  const [tags, setTags] = useState(initial?.tags?.join(", ") || "");

  const valid = title.trim().length > 2 && content.trim().length > 10;

  return (
    <div className="p-5 rounded-2xl border border-gray-200 bg-white shadow-sm">
      <div className="grid gap-4">
        <div className="grid gap-2">
          <label className="text-sm font-medium">Title</label>
          <input
            className="w-full rounded-xl border border-gray-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-black"
            placeholder="e.g. My first blog post"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
          />
        </div>
        <div className="grid gap-2">
          <label className="text-sm font-medium">Content</label>
          <textarea
            rows={8}
            className="w-full rounded-xl border border-gray-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-black"
            placeholder="Write your post here…"
            value={content}
            onChange={(e) => setContent(e.target.value)}
          />
        </div>
        <div className="grid gap-2">
          <label className="text-sm font-medium">Tags (comma separated)</label>
          <input
            className="w-full rounded-xl border border-gray-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-black"
            placeholder="books, thoughts, ideas"
            value={tags}
            onChange={(e) => setTags(e.target.value)}
          />
        </div>
        <div className="flex items-center justify-end gap-2">
          <button
            onClick={onCancel}
            className="px-4 py-2 rounded-2xl border border-gray-200 hover:bg-gray-50 inline-flex items-center gap-2"
          >
            <X className="w-4 h-4" /> Cancel
          </button>
          <button
            onClick={() => {
              const payload = {
                id: initial?.id || crypto.randomUUID(),
                slug: initial?.slug || slugify(title),
                title: title.trim(),
                content: content.trim(),
                excerpt: content.trim().slice(0, 160),
                date: initial?.date || new Date().toISOString(),
                tags: tags
                  .split(",")
                  .map((t) => t.trim())
                  .filter(Boolean),
              };
              onSave(payload);
            }}
            disabled={!valid}
            className={`px-4 py-2 rounded-2xl inline-flex items-center gap-2 ${
              valid ? "bg-black text-white" : "bg-gray-200 text-gray-600 cursor-not-allowed"
            }`}
          >
            <Save className="w-4 h-4" /> Save
          </button>
        </div>
      </div>
    </div>
  );
}

function Reader({ post, onBack }) {
  return (
    <div className="p-5 rounded-2xl border border-gray-200 bg-white shadow-sm">
      <button
        onClick={onBack}
        className="mb-4 px-3 py-1.5 rounded-xl border border-gray-200 hover:bg-gray-50"
      >
        ← Back
      </button>
      <h1 className="text-2xl font-bold">{post.title}</h1>
      <div className="flex items-center gap-3 text-sm text-gray-500 mt-1">
        <span className="inline-flex items-center gap-1">
          <Calendar className="w-4 h-4" /> {new Date(post.date).toLocaleDateString()}
        </span>
        {post.tags?.length ? (
          <span className="inline-flex items-center gap-1">
            <Tag className="w-4 h-4" /> {post.tags.join(", ")}
          </span>
        ) : null}
      </div>
      <div className="prose prose-lg max-w-none mt-6">
        <p style={{ whiteSpace: "pre-wrap" }}>{post.content}</p>
      </div>
    </div>
  );
}

export default function App() {
  const [route, setRoute] = useState("home");
  const [posts, setPosts] = useState([]);
  const [editing, setEditing] = useState(null);
  const [reading, setReading] = useState(null);

  useEffect(() => {
    setPosts(loadPosts());
  }, []);

  const sorted = useMemo(
    () => [...posts].sort((a, b) => new Date(b.date) - new Date(a.date)),
    [posts]
  );

  const startCreate = () => setEditing({});
  const startEdit = (p) => setEditing(p);
  const cancelEdit = () => setEditing(null);

  const save = (post) => {
    setPosts((prev) => {
      const exists = prev.some((p) => p.id === post.id);
      const next = exists ? prev.map((p) => (p.id === post.id ? post : p)) : [post, ...prev];
      savePosts(next);
      return next;
    });
    setEditing(null);
    setRoute("blog");
  };

  const remove = (post) => {
    if (!confirm("Are you sure you want to delete this post?")) return;
    setPosts((prev) => {
      const next = prev.filter((p) => p.id !== post.id);
      savePosts(next);
      return next;
    });
  };

  return (
    <div className="min-h-screen bg-gray-50">
      <Navbar route={typeof route === "string" ? route : "blog"} setRoute={setRoute} />

      <main className="max-w-5xl mx-auto px-4 py-8">
        {/* HOME */}
        {route === "home" && !reading && (
          <motion.section initial={{ opacity: 0 }} animate={{ opacity: 1 }}>
            <div className="grid md:grid-cols-2 gap-6 items-center">
              <div className="order-2 md:order-1">
                <h1 className="text-3xl md:text-4xl font-extrabold leading-tight">
                  Welcome to <span className="text-black">Book to Book Side</span>
                </h1>
                <p className="text-gray-600 mt-4">
                  A simple two-page blog platform. Write, save, and share your ideas — no server
                  required (everything is saved in your browser).
                </p>
                <p className="mt-2 text-sm text-gray-500">Created & Written by <strong>Jonsa</strong></p>
                <div className="flex gap-3 mt-6">
                  <button
                    onClick={() => setRoute("blog")}
                    className="px-4 py-2 rounded-2xl bg-black text-white inline-flex items-center gap-2"
                  >
                    <FileText className="w-4 h-4" /> View Blog
                  </button>
                  <button
                    onClick={startCreate}
                    className="px-4 py-2 rounded-2xl border border-gray-200 hover:bg-gray-50 inline-flex items-center gap-2"
                  >
                    <Plus className="w-4 h-4" /> New Post
                  </button>
                </div>
              </div>
              <div className="order-1 md:order-2">
                <div className="aspect-[4/3] rounded-3xl bg-white border border-gray-200 shadow-sm grid place-items-center text-center p-6">
                  <div className="text-6xl">📚</div>
                  <p className="mt-3 text-gray-600">"One post at a time — from book to book, idea to idea."</p>
                </div>
              </div>
            </div>
          </motion.section>
        )}

        {/* BLOG */}
        {route === "blog" && !reading && (
          <motion.section initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="grid gap-6">
            <div className="flex items-center justify-between">
              <h2 className="text-2xl font-bold">Blog</h2>
              <button
                onClick={startCreate}
                className="px-4 py-2 rounded-2xl bg-black text-white inline-flex items-center gap-2"
              >
                <Plus className="w-4 h-4" /> New Post
              </button>
            </div>

            {editing ? (
              <PostForm initial={editing} onCancel={cancelEdit} onSave={save} />
            ) : sorted.length === 0 ? (
              <EmptyState onCreate={startCreate} />
            ) : (
              <div className="grid gap-4">
                {sorted.map((post) => (
                  <PostCard
                    key={post.id}
                    post={post}
                    onEdit={startEdit}
                    onDelete={remove}
                    onOpen={(p) => setReading(p)}
                  />
                ))}
              </div>
            )}
          </motion.section>
        )}

        {/* READER */}
        {reading && <Reader post={reading} onBack={() => setReading(null)} />}
      </main>

      <footer className="mt-10 border-t border-gray-200">
        <div className="max-w-5xl mx-auto px-4 py-6 text-sm text-gray-600 flex flex-col md:flex-row items-center justify-between gap-3">
          <div>
            © {new Date().getFullYear()} Book to Book Side — Written by <strong>Jonsa</strong>. All rights reserved.
          </div>
          <div className="flex items-center gap-3">
            <a
              className="hover:underline"
              href="#"
              onClick={(e) => {
                e.preventDefault();
                setRoute("home");
              }}
            >
              Home
            </a>
            <span>•</span>
            <a
              className="hover:underline"
              href="#"
              onClick={(e) => {
                e.preventDefault();
                setRoute("blog");
              }}
            >
              Blog
            </a>
          </div>
        </div>
      </footer>
    </div>
  );
}
