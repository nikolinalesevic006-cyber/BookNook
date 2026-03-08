# BookNook
 BookNook platforma ima zadatak da pruži najjednostavniji mogući način za razmenu studentskih knjiga. 

import { useState, useEffect } from "react";
import { motion } from "framer-motion";
import { BookOpen, GraduationCap, ShoppingBag, Tag, ArrowRight } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Navbar } from "@/components/Navbar";
import { BookCard } from "@/components/BookCard";
import { supabase } from "@/integrations/supabase/client";
import { Tables } from "@/integrations/supabase/types";
import { useNavigate } from "react-router-dom";
import { useAuth } from "@/lib/auth-context";
import { useSavedBooks } from "@/hooks/useSavedBooks";
import heroImage from "@/assets/hero-books.jpg";

export default function Index() {
  const [latestBooks, setLatestBooks] = useState<Tables<"books">[]>([]);
  const [loading, setLoading] = useState(true);
  const navigate = useNavigate();
  const { user } = useAuth();
  const { savedIds, toggleSave } = useSavedBooks();

  useEffect(() => {
    const fetchLatest = async () => {
      const { data } = await supabase
        .from("books")
        .select("*")
        .eq("is_sold", false)
        .order("created_at", { ascending: false })
        .limit(8);
      setLatestBooks(data || []);
      setLoading(false);
    };
    fetchLatest();
  }, []);

  const handleCTA = (intent: "buy" | "sell") => {
    if (intent === "buy") {
      navigate("/shop"); // Anyone can browse books
    } else {
      // Selling requires authentication
      if (user) {
        navigate("/sell");
      } else {
        navigate("/auth?intent=sell");
      }
    }
  };

  return (
    <div className="min-h-screen bg-background">
      <Navbar />

      {/* Hero */}
      <section className="relative overflow-hidden gradient-hero py-24 md:py-32">
        <div className="absolute inset-0 opacity-20">
          <img src={heroImage} alt="" className="h-full w-full object-cover" />
        </div>
        <div className="container relative mx-auto px-4 text-center">
          <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 0.6 }}>
            <div className="mx-auto mb-6 flex h-16 w-16 items-center justify-center rounded-2xl gradient-primary shadow-primary">
              <BookOpen className="h-8 w-8 text-primary-foreground" />
            </div>
            <h1 className="heading-display mb-4 text-primary-foreground md:text-5xl lg:text-6xl">
              BookNook
            </h1>
            <p className="mx-auto mb-10 max-w-2xl text-lg text-primary-foreground/80">
              Marketplace za studentsku literaturu. Pronađi ili prodaj udžbenike brzo i jednostavno.
            </p>
            <div className="flex flex-col sm:flex-row justify-center gap-4">
              <Button
                size="lg"
                onClick={() => handleCTA("buy")}
                className="gradient-primary border-0 shadow-primary text-base px-8 py-6"
              >
                <ShoppingBag className="h-5 w-5 mr-2" />
                Kupi knjigu
              </Button>
              <Button
                size="lg"
                onClick={() => handleCTA("sell")}
                className="bg-primary/20 border border-primary-foreground/20 text-primary-foreground hover:bg-primary/40 text-base px-8 py-6 backdrop-blur-sm"
              >
                <Tag className="h-5 w-5 mr-2" />
                Prodaj knjigu
              </Button>
            </div>
            <div className="mt-6">
              <Button
                variant="ghost"
                size="sm"
                onClick={() => navigate("/chatbot")}
                className="text-primary-foreground/70 hover:text-primary-foreground hover:bg-primary-foreground/10"
              >
                AI Preporuke →
              </Button>
            </div>
          </motion.div>
        </div>
      </section>

      {/* Stats */}
      <section className="border-b border-border bg-card py-8">
        <div className="container mx-auto flex items-center justify-center px-4">
          <div className="flex items-center gap-3 rounded-xl bg-secondary/50 px-8 py-4">
            <div className="flex h-10 w-10 items-center justify-center rounded-lg gradient-primary">
              <GraduationCap className="h-5 w-5 text-primary-foreground" />
            </div>
            <div>
              <p className="font-display text-2xl font-bold text-foreground">12+</p>
              <p className="text-sm text-muted-foreground">Fakulteta širom Srbije</p>
            </div>
          </div>
        </div>
      </section>

      {/* Latest Books */}
      <section className="container mx-auto px-4 py-14">
        <div className="flex items-center justify-between mb-8">
          <div>
            <h2 className="heading-medium text-foreground">Najnovije knjige</h2>
            <p className="text-sm text-muted-foreground mt-2">Poslednje dodati udžbenici na platformi</p>
          </div>
          <Button variant="outline" onClick={() => navigate("/shop")} className="hidden sm:flex">
            Sve knjige <ArrowRight className="h-4 w-4 ml-2" />
          </Button>
        </div>

        {loading ? (
          <div className="grid gap-5 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
            {[...Array(8)].map((_, i) => (
              <div key={i} className="animate-pulse rounded-xl bg-muted">
                <div className="aspect-[3/4] rounded-t-xl bg-muted-foreground/10" />
                <div className="p-4 space-y-2">
                  <div className="h-4 w-3/4 rounded bg-muted-foreground/10" />
                  <div className="h-3 w-1/2 rounded bg-muted-foreground/10" />
                  <div className="h-6 w-1/3 rounded bg-muted-foreground/10 mt-3" />
                </div>
              </div>
            ))}
          </div>
        ) : latestBooks.length === 0 ? (
          <div className="py-16 text-center">
            <BookOpen className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
            <h3 className="mb-2 font-display text-lg font-semibold text-foreground">Još nema knjiga</h3>
            <p className="text-sm text-muted-foreground">Budite prvi koji će dodati udžbenik</p>
            <Button className="mt-4" onClick={() => handleCTA("sell")}>Dodaj prvu knjigu</Button>
          </div>
        ) : (
          <div className="grid gap-5 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
            {latestBooks.map((book) => (
              <BookCard key={book.id} book={book} isSaved={savedIds.has(book.id)} onToggleSave={toggleSave} />
            ))}
          </div>
        )}

        <div className="mt-8 text-center sm:hidden">
          <Button variant="outline" onClick={() => navigate("/shop")} className="w-full">
            Pregledaj sve knjige <ArrowRight className="h-4 w-4 ml-2" />
          </Button>
        </div>
      </section>
    </div>
  );
}
import { useState } from "react";
import { Loader2 } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { FacultySelect } from "@/components/FacultySelect";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogFooter,
} from "@/components/ui/dialog";

interface EditProfileDialogProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  userId: string;
  fullName: string | null;
  faculty: string | null;
  onSave: (data: { fullName: string; faculty: string }) => void;
}

export function EditProfileDialog({ open, onOpenChange, userId, fullName, faculty, onSave }: EditProfileDialogProps) {
  const [name, setName] = useState(fullName || "");
  const [fac, setFac] = useState(faculty || "");
  const [saving, setSaving] = useState(false);

  const handleSave = async () => {
    if (!name.trim()) {
      toast.error("Ime je obavezno");
      return;
    }
    setSaving(true);
    const { error } = await supabase
      .from("profiles")
      .update({ full_name: name.trim(), faculty: fac || null })
      .eq("user_id", userId);
    setSaving(false);

    if (error) {
      toast.error("Greška pri čuvanju profila");
      return;
    }
    toast.success("Profil ažuriran");
    onSave({ fullName: name.trim(), faculty: fac });
    onOpenChange(false);
  };

  // Reset fields when dialog opens
  const handleOpenChange = (v: boolean) => {
    if (v) {
      setName(fullName || "");
      setFac(faculty || "");
    }
    onOpenChange(v);
  };

  return (
    <Dialog open={open} onOpenChange={handleOpenChange}>
      <DialogContent className="sm:max-w-md">
        <DialogHeader>
          <DialogTitle>Izmijeni profil</DialogTitle>
        </DialogHeader>
        <div className="space-y-4 py-2">
          <div className="space-y-2">
            <Label htmlFor="edit-name">Ime i prezime</Label>
            <Input
              id="edit-name"
              value={name}
              onChange={(e) => setName(e.target.value)}
              placeholder="Vaše ime"
            />
          </div>
          <div className="space-y-2">
            <Label>Fakultet</Label>
            <FacultySelect value={fac} onValueChange={setFac} />
          </div>
        </div>
        <DialogFooter>
          <Button variant="outline" onClick={() => onOpenChange(false)}>
            Otkaži
          </Button>
          <Button onClick={handleSave} disabled={saving}>
            {saving && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
            Sačuvaj
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
import { useState, useRef } from "react";
import { Camera, X, Loader2, Mail, GraduationCap } from "lucide-react";
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import { Button } from "@/components/ui/button";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";

interface ProfileHeaderProps {
  userId: string;
  email: string;
  fullName: string | null;
  faculty: string | null;
  avatarUrl: string | null;
  onAvatarChange: (url: string | null) => void;
  onEditClick: () => void;
}

export function ProfileHeader({ userId, email, fullName, faculty, avatarUrl, onAvatarChange, onEditClick }: ProfileHeaderProps) {
  const [uploading, setUploading] = useState(false);
  const inputRef = useRef<HTMLInputElement>(null);

  const initials = (fullName || email || "U")
    .split(" ")
    .map((w) => w[0])
    .join("")
    .toUpperCase()
    .slice(0, 2);

  const handleUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;
    if (file.size > 2 * 1024 * 1024) {
      toast.error("Maksimalna veličina slike je 2MB");
      return;
    }

    setUploading(true);
    try {
      const ext = file.name.split(".").pop();
      const path = `${userId}/avatar.${ext}`;

      // Remove old avatar files
      const { data: existing } = await supabase.storage.from("avatars").list(userId);
      if (existing && existing.length > 0) {
        await supabase.storage.from("avatars").remove(existing.map((f) => `${userId}/${f.name}`));
      }

      const { error: uploadError } = await supabase.storage.from("avatars").upload(path, file, { upsert: true });
      if (uploadError) throw uploadError;

      const { data: urlData } = supabase.storage.from("avatars").getPublicUrl(path);
      const newUrl = `${urlData.publicUrl}?t=${Date.now()}`;

      await supabase.from("profiles").update({ avatar_url: newUrl }).eq("user_id", userId);
      onAvatarChange(newUrl);
      toast.success("Profilna slika ažurirana");
    } catch (err) {
      console.error(err);
      toast.error("Greška pri otpremanju slike");
    } finally {
      setUploading(false);
      if (inputRef.current) inputRef.current.value = "";
    }
  };

  const handleRemove = async () => {
    setUploading(true);
    try {
      const { data: existing } = await supabase.storage.from("avatars").list(userId);
      if (existing && existing.length > 0) {
        await supabase.storage.from("avatars").remove(existing.map((f) => `${userId}/${f.name}`));
      }
      await supabase.from("profiles").update({ avatar_url: null }).eq("user_id", userId);
      onAvatarChange(null);
      toast.success("Profilna slika uklonjena");
    } catch {
      toast.error("Greška pri uklanjanju slike");
    } finally {
      setUploading(false);
    }
  };

  return (
    <div className="rounded-2xl border border-border bg-card p-6 shadow-card sm:p-8">
      <div className="flex flex-col items-center gap-6 sm:flex-row sm:items-start">
        {/* Avatar */}
        <div className="group relative">
          <Avatar className="h-24 w-24 ring-4 ring-primary/20 sm:h-28 sm:w-28">
            <AvatarImage src={avatarUrl || undefined} alt={fullName || "Avatar"} />
            <AvatarFallback className="bg-primary/10 text-2xl font-bold text-primary">
              {initials}
            </AvatarFallback>
          </Avatar>
          <div className="absolute inset-0 flex items-center justify-center gap-1 rounded-full bg-foreground/40 opacity-0 transition-opacity group-hover:opacity-100">
            <button
              type="button"
              onClick={() => inputRef.current?.click()}
              disabled={uploading}
              className="rounded-full bg-card p-1.5 text-foreground shadow-md transition hover:bg-primary hover:text-primary-foreground"
            >
              {uploading ? <Loader2 className="h-4 w-4 animate-spin" /> : <Camera className="h-4 w-4" />}
            </button>
            {avatarUrl && (
              <button
                type="button"
                onClick={handleRemove}
                disabled={uploading}
                className="rounded-full bg-card p-1.5 text-destructive shadow-md transition hover:bg-destructive hover:text-destructive-foreground"
              >
                <X className="h-4 w-4" />
              </button>
            )}
          </div>
          <input ref={inputRef} type="file" accept="image/*" onChange={handleUpload} className="hidden" />
        </div>

        {/* Info */}
        <div className="flex-1 text-center sm:text-left">
          <h1 className="font-display text-2xl font-bold text-foreground sm:text-3xl">
            {fullName || "Korisnik"}
          </h1>
          {faculty && (
            <p className="mt-1 flex items-center justify-center gap-1.5 text-sm text-muted-foreground sm:justify-start">
              <GraduationCap className="h-4 w-4" />
              {faculty}
            </p>
          )}
          <p className="mt-1 flex items-center justify-center gap-1.5 text-xs text-muted-foreground/70 sm:justify-start">
            <Mail className="h-3.5 w-3.5" />
            {email}
          </p>
          <Button variant="outline" size="sm" className="mt-4" onClick={onEditClick}>
            Izmeni profil
          </Button>
        </div>
      </div>
    </div>
  );
}
import { BookOpen, ShoppingBag, Heart } from "lucide-react";

interface ProfileStatsProps {
  activeListings: number;
  soldBooks: number;
  savedBooks: number;
}

export function ProfileStats({ activeListings, soldBooks, savedBooks }: ProfileStatsProps) {
  const stats = [
    { label: "Aktivni oglasi", value: activeListings, icon: BookOpen, color: "text-primary" },
    { label: "Prodato", value: soldBooks, icon: ShoppingBag, color: "text-emerald-600" },
    { label: "Sačuvano", value: savedBooks, icon: Heart, color: "text-destructive" },
  ];

  return (
    <div className="grid grid-cols-3 gap-3">
      {stats.map((s) => (
        <div
          key={s.label}
          className="flex flex-col items-center gap-1.5 rounded-xl border border-border bg-card p-4 shadow-card"
        >
          <s.icon className={`h-5 w-5 ${s.color}`} />
          <span className="font-display text-2xl font-bold text-foreground">{s.value}</span>
          <span className="text-xs text-muted-foreground">{s.label}</span>
        </div>
      ))}
    </div>
  );
}
import { BookOpen, Heart } from "lucide-react";
import { Card, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Tables } from "@/integrations/supabase/types";
import { Link } from "react-router-dom";

interface BookCardProps {
  book: Tables<"books">;
  isSaved?: boolean;
  onToggleSave?: (bookId: string) => void;
}

const conditionColors: Record<string, string> = {
  novo: "bg-green-100 text-green-800 border-green-200",
  "kao novo": "bg-blue-100 text-blue-800 border-blue-200",
  dobro: "bg-yellow-100 text-yellow-800 border-yellow-200",
  korisceno: "bg-orange-100 text-orange-800 border-orange-200",
};

export function BookCard({ book, isSaved, onToggleSave }: BookCardProps) {
  return (
    <Link to={`/listing/${book.id}`} className="block group">
      <Card className="overflow-hidden border-border/60 bg-card shadow-card transition-all duration-300 hover:shadow-card-hover hover:-translate-y-1.5 hover:border-primary/30">
        <div className="relative aspect-[3/4] overflow-hidden bg-secondary">
          {book.image_url ? (
            <img src={book.image_url} alt={book.title} className="h-full w-full object-cover transition-transform duration-500 group-hover:scale-110" />
          ) : (
            <div className="flex h-full w-full items-center justify-center bg-gradient-to-br from-secondary to-accent/30">
              <BookOpen className="h-12 w-12 text-primary/30" />
            </div>
          )}
          {book.is_sold && (
            <div className="absolute inset-0 flex items-center justify-center bg-foreground/50 backdrop-blur-sm">
              <span className="rounded-lg bg-destructive px-4 py-1.5 font-display text-sm font-bold text-destructive-foreground">PRODATO</span>
            </div>
          )}
          <div className="absolute top-2 right-2 flex items-center gap-1.5">
            {onToggleSave && (
              <button
                onClick={(e) => { e.preventDefault(); e.stopPropagation(); onToggleSave(book.id); }}
                className="flex h-8 w-8 items-center justify-center rounded-full bg-card/80 backdrop-blur-sm shadow-md transition-all hover:scale-110"
                title={isSaved ? "Ukloni iz sačuvanih" : "Sačuvaj"}
              >
                <Heart className={`h-4 w-4 transition-colors ${isSaved ? "fill-destructive text-destructive" : "text-muted-foreground"}`} />
              </button>
            )}
            <span className={`rounded-full border px-2.5 py-0.5 text-[10px] font-semibold ${conditionColors[book.condition] || "bg-muted text-muted-foreground"}`}>
              {book.condition}
            </span>
          </div>
        </div>
        <CardContent className="p-4">
          <h3 className="mb-1 font-display text-sm font-semibold leading-tight text-foreground line-clamp-2 group-hover:text-primary transition-colors">
            {book.title}
          </h3>
          <p className="mb-3 text-xs text-muted-foreground">{book.author}</p>
          <div className="mb-3 flex flex-wrap gap-1">
            <Badge variant="secondary" className="text-[10px] font-medium">{book.faculty}</Badge>
            <Badge variant="outline" className="text-[10px]">{book.year_of_study}. godina</Badge>
          </div>
          <div className="flex items-center justify-between border-t border-border/50 pt-3">
            <span className="font-display text-lg font-bold text-primary">{book.price} RSD</span>
          </div>
        </CardContent>
      </Card>
    </Link>
  );
}
import { useState, useEffect, useMemo } from "react";
import { Check, ChevronsUpDown, Plus, Search } from "lucide-react";
import { Popover, PopoverContent, PopoverTrigger } from "@/components/ui/popover";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { cn } from "@/lib/utils";
import { supabase } from "@/integrations/supabase/client";

interface FacultySelectProps {
  value: string;
  onValueChange: (value: string) => void;
  placeholder?: string;
  disabled?: boolean;
  className?: string;
}

export function FacultySelect({ value, onValueChange, placeholder = "Izaberite fakultet", disabled, className }: FacultySelectProps) {
  const [open, setOpen] = useState(false);
  const [faculties, setFaculties] = useState<string[]>([]);
  const [search, setSearch] = useState("");
  const [showNewInput, setShowNewInput] = useState(false);
  const [newFaculty, setNewFaculty] = useState("");
  const [adding, setAdding] = useState(false);

  useEffect(() => {
    const load = async () => {
      const { data } = await supabase.from("faculties").select("name").order("name");
      if (data) setFaculties(data.map((f) => f.name));
    };
    load();
  }, []);

  const filtered = useMemo(() => {
    if (!search) return faculties;
    const q = search.toLowerCase();
    return faculties.filter((f) => f.toLowerCase().includes(q));
  }, [faculties, search]);

  const handleAddNew = async () => {
    const trimmed = newFaculty.trim();
    if (!trimmed) return;

    // Check case-insensitive duplicate
    if (faculties.some((f) => f.toLowerCase() === trimmed.toLowerCase())) {
      const existing = faculties.find((f) => f.toLowerCase() === trimmed.toLowerCase())!;
      onValueChange(existing);
      setShowNewInput(false);
      setNewFaculty("");
      setOpen(false);
      return;
    }

    setAdding(true);
    const { error } = await supabase.from("faculties").insert({ name: trimmed });
    setAdding(false);

    if (!error) {
      setFaculties((prev) => [...prev, trimmed].sort((a, b) => a.localeCompare(b)));
      onValueChange(trimmed);
      setShowNewInput(false);
      setNewFaculty("");
      setOpen(false);
    }
  };

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button
          variant="outline"
          role="combobox"
          aria-expanded={open}
          disabled={disabled}
          className={cn(
            "w-full justify-between font-normal",
            !value && "text-muted-foreground",
            className,
          )}
        >
          <span className="truncate">{value || placeholder}</span>
          <ChevronsUpDown className="ml-2 h-4 w-4 shrink-0 opacity-50" />
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-[--radix-popover-trigger-width] p-0" align="start">
        <div className="p-2">
          <div className="relative">
            <Search className="absolute left-2.5 top-1/2 h-3.5 w-3.5 -translate-y-1/2 text-muted-foreground" />
            <Input
              placeholder="Pretraži fakultete..."
              value={search}
              onChange={(e) => setSearch(e.target.value)}
              className="pl-8 h-9 text-sm"
            />
          </div>
        </div>

        <div className="max-h-[200px] overflow-y-auto px-1 pb-1">
          {filtered.length === 0 && (
            <p className="py-3 text-center text-sm text-muted-foreground">Nema rezultata</p>
          )}
          {filtered.map((f) => (
            <button
              key={f}
              type="button"
              onClick={() => { onValueChange(f); setOpen(false); setSearch(""); }}
              className={cn(
                "flex w-full items-center gap-2 rounded-sm px-2 py-1.5 text-sm hover:bg-accent hover:text-accent-foreground cursor-pointer",
                value === f && "bg-accent text-accent-foreground",
              )}
            >
              <Check className={cn("h-3.5 w-3.5 shrink-0", value === f ? "opacity-100" : "opacity-0")} />
              <span className="truncate">{f}</span>
            </button>
          ))}
        </div>

        <div className="border-t p-2">
          {!showNewInput ? (
            <button
              type="button"
              onClick={() => setShowNewInput(true)}
              className="flex w-full items-center gap-2 rounded-sm px-2 py-1.5 text-sm text-primary hover:bg-accent cursor-pointer"
            >
              <Plus className="h-3.5 w-3.5" />
              Dodaj novi fakultet
            </button>
          ) : (
            <div className="flex gap-2">
              <Input
                placeholder="Naziv fakulteta"
                value={newFaculty}
                onChange={(e) => setNewFaculty(e.target.value)}
                onKeyDown={(e) => e.key === "Enter" && (e.preventDefault(), handleAddNew())}
                className="h-9 text-sm flex-1"
                autoFocus
              />
              <Button size="sm" onClick={handleAddNew} disabled={adding || !newFaculty.trim()}>
                {adding ? "..." : "Dodaj"}
              </Button>
            </div>
          )}
        </div>
      </PopoverContent>
    </Popover>
  );
}
import { useState, useRef } from "react";
import { ImagePlus, X, Loader2 } from "lucide-react";
import { cn } from "@/lib/utils";

interface ImageUploadProps {
  images: File[];
  previews: string[];
  onAdd: (files: File[]) => void;
  onRemove: (index: number) => void;
  maxImages?: number;
  disabled?: boolean;
}

export function ImageUpload({ images, previews, onAdd, onRemove, maxImages = 5, disabled }: ImageUploadProps) {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleFiles = (e: React.ChangeEvent<HTMLInputElement>) => {
    const files = Array.from(e.target.files || []);
    const remaining = maxImages - images.length;
    if (remaining <= 0) return;
    onAdd(files.slice(0, remaining));
    if (inputRef.current) inputRef.current.value = "";
  };

  const canAdd = images.length < maxImages && !disabled;

  return (
    <div className="space-y-2">
      <div className="grid grid-cols-3 gap-3 sm:grid-cols-5">
        {previews.map((src, i) => (
          <div key={i} className="group relative aspect-square overflow-hidden rounded-lg border border-border bg-muted">
            <img src={src} alt={`Slika ${i + 1}`} className="h-full w-full object-cover" />
            {!disabled && (
              <button
                type="button"
                onClick={() => onRemove(i)}
                className="absolute right-1 top-1 rounded-full bg-destructive p-1 text-destructive-foreground opacity-0 transition-opacity group-hover:opacity-100"
              >
                <X className="h-3 w-3" />
              </button>
            )}
          </div>
        ))}

        {canAdd && (
          <button
            type="button"
            onClick={() => inputRef.current?.click()}
            className={cn(
              "flex aspect-square flex-col items-center justify-center gap-1 rounded-lg border-2 border-dashed border-muted-foreground/30 bg-muted/50 text-muted-foreground transition-colors hover:border-primary hover:text-primary"
            )}
          >
            <ImagePlus className="h-6 w-6" />
            <span className="text-[10px] font-medium">{images.length}/{maxImages}</span>
          </button>
        )}
      </div>

      <input
        ref={inputRef}
        type="file"
        accept="image/*"
        multiple
        onChange={handleFiles}
        className="hidden"
      />
      <p className="text-xs text-muted-foreground">Dodajte do {maxImages} fotografija knjige (JPG, PNG)</p>
    </div>
  );
}
import { useState, useEffect } from "react";
import { BookOpen, LogOut, Plus, User, MessageCircle, ShoppingBag, Shield, FileText, Mail, HelpCircle } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { useAuth } from "@/lib/auth-context";
import { useAdmin } from "@/hooks/useAdmin";
import { supabase } from "@/integrations/supabase/client";
import { Link, useNavigate } from "react-router-dom";

export function Navbar() {
  const { user, signOut } = useAuth();
  const { isAdmin } = useAdmin();
  const navigate = useNavigate();
  const [unreadCount, setUnreadCount] = useState(0);

  useEffect(() => {
    if (!user) { setUnreadCount(0); return; }
    fetchUnread();

    const channel = supabase
      .channel("navbar-unread")
      .on("postgres_changes", { event: "INSERT", schema: "public", table: "messages" }, () => {
        fetchUnread();
      })
      .on("postgres_changes", { event: "UPDATE", schema: "public", table: "messages" }, () => {
        fetchUnread();
      })
      .subscribe();

    return () => { supabase.removeChannel(channel); };
  }, [user]);

  const fetchUnread = async () => {
    if (!user) return;
    const { data: convos } = await supabase
      .from("conversations")
      .select("id");

    if (!convos || convos.length === 0) { setUnreadCount(0); return; }

    const ids = convos.map((c: any) => c.id);
    const { count } = await supabase
      .from("messages")
      .select("id", { count: "exact", head: true })
      .in("conversation_id", ids)
      .eq("is_read", false)
      .neq("sender_id", user.id);

    setUnreadCount(count || 0);
  };

  return (
    <nav className="sticky top-0 z-50 border-b border-border bg-card/80 backdrop-blur-md w-full">
      <div className="container mx-auto flex h-16 items-center justify-between px-4 max-w-7xl">
        <Link to="/" className="flex items-center gap-2.5">
          <div className="flex h-9 w-9 items-center justify-center rounded-lg gradient-primary">
            <BookOpen className="h-5 w-5 text-primary-foreground" />
          </div>
          <span className="font-display text-lg font-bold tracking-tight text-foreground">BookNook</span>
        </Link>

        <div className="flex items-center gap-1 flex-shrink-0">
          <Button variant="ghost" size="sm" onClick={() => navigate("/shop")} className="nav-link hover:text-primary-dark transition-colors">
            <ShoppingBag className="h-4 w-4 mr-1.5" />
            <span className="hidden sm:inline">Prodavnica</span>
          </Button>
          <Button variant="ghost" size="sm" onClick={() => navigate("/materials")} className="nav-link hover:text-primary-dark transition-colors">
            <FileText className="h-4 w-4 mr-1.5" />
            <span className="hidden sm:inline">Materijali</span>
          </Button>
          <Button variant="ghost" size="sm" onClick={() => navigate("/chatbot")} className="hidden sm:flex nav-link hover:text-primary-dark transition-colors">
            AI Pomoćnik
          </Button>
          <Button variant="ghost" size="sm" onClick={() => navigate("/contact")} className="nav-link hover:text-primary-dark transition-colors">
            <Mail className="h-4 w-4 mr-1.5" />
            <span className="hidden sm:inline">Kontakt</span>
          </Button>
          {user ? (
            <>
              <Button variant="outline" size="sm" onClick={() => navigate("/sell")} className="hidden sm:flex nav-link hover:text-primary-dark hover:border-primary-dark transition-colors">
                <Plus className="h-4 w-4 mr-1.5" />
                Prodaj
              </Button>
              <Button variant="ghost" size="icon" className="relative" onClick={() => navigate("/inbox")} title="Poruke">
                <MessageCircle className="h-4 w-4" />
                {unreadCount > 0 && (
                  <Badge className="absolute -right-1 -top-1 h-5 min-w-5 rounded-full px-1 text-[10px] flex items-center justify-center">
                    {unreadCount > 99 ? "99+" : unreadCount}
                  </Badge>
                )}
              </Button>
              {!isAdmin && (
                <Button variant="ghost" size="icon" onClick={() => navigate("/support")} title="Podrška">
                  <HelpCircle className="h-4 w-4" />
                </Button>
              )}
              {isAdmin && (
                <Button variant="ghost" size="icon" onClick={() => navigate("/admin")} title="Admin">
                  <Shield className="h-4 w-4" />
                </Button>
              )}
              <Button variant="ghost" size="icon" onClick={() => navigate("/profile")}>
                <User className="h-4 w-4" />
              </Button>
              <Button variant="ghost" size="icon" onClick={() => signOut()}>
                <LogOut className="h-4 w-4" />
              </Button>
            </>
          ) : (
            <Button size="sm" className="button-text" onClick={() => navigate("/auth")}>
              Prijavi se
            </Button>
          )}
        </div>
      </div>
    </nav>
  );
}
import { NavLink as RouterNavLink, NavLinkProps } from "react-router-dom";
import { forwardRef } from "react";
import { cn } from "@/lib/utils";

interface NavLinkCompatProps extends Omit<NavLinkProps, "className"> {
  className?: string;
  activeClassName?: string;
  pendingClassName?: string;
}

const NavLink = forwardRef<HTMLAnchorElement, NavLinkCompatProps>(
  ({ className, activeClassName, pendingClassName, to, ...props }, ref) => {
    return (
      <RouterNavLink
        ref={ref}
        to={to}
        className={({ isActive, isPending }) =>
          cn(className, isActive && activeClassName, isPending && pendingClassName)
        }
        {...props}
      />
    );
  },
);

NavLink.displayName = "NavLink";

export { NavLink };import { useState } from "react";
import { supabase } from "@/integrations/supabase/client";
import { useAuth } from "@/lib/auth-context";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from "@/components/ui/dialog";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Textarea } from "@/components/ui/textarea";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";
import { Flag, Loader2 } from "lucide-react";
import { toast } from "sonner";

const REASONS = [
  { value: "already_sold", label: "Već prodato" },
  { value: "incorrect", label: "Netačne informacije" },
  { value: "spam", label: "Spam" },
  { value: "inappropriate", label: "Neprikladni sadržaj" },
  { value: "other", label: "Ostalo" },
];

interface Props {
  bookId: string;
}

export function ReportListingDialog({ bookId }: Props) {
  const { user } = useAuth();
  const [open, setOpen] = useState(false);
  const [reason, setReason] = useState("");
  const [description, setDescription] = useState("");
  const [submitting, setSubmitting] = useState(false);

  if (!user) return null;

  const handleSubmit = async () => {
    if (!reason) {
      toast.error("Izaberite razlog prijave");
      return;
    }
    setSubmitting(true);
    const { error } = await supabase.from("reports").insert({
      book_id: bookId,
      user_id: user.id,
      reason,
      description: description.trim() || null,
    });
    setSubmitting(false);
    if (error) {
      toast.error("Greška pri slanju prijave");
      return;
    }
    toast.success("Prijava je poslata. Hvala!");
    setOpen(false);
    setReason("");
    setDescription("");
  };

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button variant="ghost" size="sm" className="text-muted-foreground hover:text-destructive">
          <Flag className="h-4 w-4 mr-1" />
          Prijavi oglas
        </Button>
      </DialogTrigger>
      <DialogContent className="sm:max-w-md">
        <DialogHeader>
          <DialogTitle>Prijavi oglas</DialogTitle>
        </DialogHeader>
        <div className="space-y-4 pt-2">
          <div className="space-y-2">
            <Label>Razlog prijave</Label>
            <Select value={reason} onValueChange={setReason}>
              <SelectTrigger>
                <SelectValue placeholder="Izaberite razlog..." />
              </SelectTrigger>
              <SelectContent>
                {REASONS.map((r) => (
                  <SelectItem key={r.value} value={r.value}>{r.label}</SelectItem>
                ))}
              </SelectContent>
            </Select>
          </div>
          <div className="space-y-2">
            <Label>Dodatni opis (opciono)</Label>
            <Textarea
              value={description}
              onChange={(e) => setDescription(e.target.value)}
              placeholder="Opišite problem..."
              rows={3}
              maxLength={500}
            />
          </div>
          <Button onClick={handleSubmit} disabled={submitting} className="w-full">
            {submitting ? <Loader2 className="mr-2 h-4 w-4 animate-spin" /> : <Flag className="mr-2 h-4 w-4" />}
            Pošalji prijavu
          </Button>
        </div>
      </DialogContent>
    </Dialog>
  );
}
import { useState } from "react";
import { Star, Loader2 } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";

interface ReviewDialogProps {
  sellerId: string;
  reviewerId: string;
  bookId: string;
  conversationId: string;
  onReviewSubmitted: () => void;
}

export function ReviewDialog({ sellerId, reviewerId, bookId, conversationId, onReviewSubmitted }: ReviewDialogProps) {
  const [open, setOpen] = useState(false);
  const [rating, setRating] = useState(0);
  const [hover, setHover] = useState(0);
  const [comment, setComment] = useState("");
  const [sending, setSending] = useState(false);

  const handleSubmit = async () => {
    if (rating === 0) {
      toast.error("Izaberite ocenu");
      return;
    }
    if (!comment.trim()) {
      toast.error("Napišite komentar");
      return;
    }
    if (comment.trim().length > 500) {
      toast.error("Komentar ne sme biti duži od 500 karaktera");
      return;
    }

    setSending(true);
    const { error } = await supabase.from("reviews").insert({
      seller_id: sellerId,
      reviewer_id: reviewerId,
      book_id: bookId,
      conversation_id: conversationId,
      rating,
      comment: comment.trim(),
    });

    if (error) {
      if (error.code === "23505") {
        toast.error("Već ste ostavili recenziju za ovaj razgovor");
      } else {
        toast.error("Greška pri slanju recenzije");
        console.error(error);
      }
    } else {
      toast.success("Recenzija je uspešno poslata!");
      onReviewSubmitted();
      setOpen(false);
      setRating(0);
      setComment("");
    }
    setSending(false);
  };

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button variant="outline" size="sm" className="gap-1.5">
          <Star className="h-4 w-4 text-yellow-500" />
          Oceni prodavca
        </Button>
      </DialogTrigger>
      <DialogContent className="sm:max-w-md">
        <DialogHeader>
          <DialogTitle className="font-display">Ocenite prodavca</DialogTitle>
        </DialogHeader>
        <div className="space-y-4 pt-2">
          {/* Stars */}
          <div className="flex justify-center gap-1">
            {[1, 2, 3, 4, 5].map((star) => (
              <button
                key={star}
                type="button"
                onClick={() => setRating(star)}
                onMouseEnter={() => setHover(star)}
                onMouseLeave={() => setHover(0)}
                className="rounded p-1 transition-transform hover:scale-110"
              >
                <Star
                  className={`h-8 w-8 transition-colors ${
                    star <= (hover || rating)
                      ? "fill-yellow-400 text-yellow-400"
                      : "text-muted-foreground/30"
                  }`}
                />
              </button>
            ))}
          </div>
          <p className="text-center text-sm text-muted-foreground">
            {rating === 0 ? "Kliknite da ocenite" : `${rating} od 5 zvezdica`}
          </p>

          {/* Comment */}
          <Textarea
            value={comment}
            onChange={(e) => setComment(e.target.value)}
            placeholder="Napišite komentar o iskustvu sa prodavcem..."
            rows={3}
            maxLength={500}
            className="resize-none"
          />
          <p className="text-right text-xs text-muted-foreground">{comment.length}/500</p>

          <Button onClick={handleSubmit} disabled={sending} className="w-full">
            {sending ? <Loader2 className="mr-2 h-4 w-4 animate-spin" /> : <Star className="mr-2 h-4 w-4" />}
            Pošalji recenziju
          </Button>
        </div>
      </DialogContent>
    </Dialog>
  );
}
import { useEffect } from "react";
import { useLocation, useNavigationType } from "react-router-dom";

export const ScrollToTop = () => {
  const { pathname } = useLocation();
  const navType = useNavigationType();

  useEffect(() => {
    if (navType !== "POP") {
      window.scrollTo({ top: 0, left: 0, behavior: "instant" });
    }
  }, [pathname, navType]);

  return null;
};
import { Search, Filter, X } from "lucide-react";
import { Input } from "@/components/ui/input";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Button } from "@/components/ui/button";
import { FacultySelect } from "@/components/FacultySelect";

interface FiltersState {
  search: string;
  faculty: string;
  year: string;
  condition: string;
}

interface Props {
  filters: FiltersState;
  onChange: (filters: FiltersState) => void;
}

export function SearchAndFilters({ filters, onChange }: Props) {
  const update = (key: keyof FiltersState, value: string) => onChange({ ...filters, [key]: value });
  const hasFilters = filters.faculty || filters.year || filters.condition;

  const clearFilters = () => onChange({ ...filters, faculty: "", year: "", condition: "" });

  return (
    <div className="space-y-4">
      <div className="relative">
        <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-primary-dark" />
        <Input
          placeholder="Pretraži po naslovu, autoru ili predmetu..."
          value={filters.search}
          onChange={(e) => update("search", e.target.value)}
          className="pl-10 h-12 text-sm bg-card border-[hsl(var(--primary-dark))] text-primary-dark placeholder:text-[hsl(var(--primary-dark))]/60 focus-visible:ring-2 focus-visible:ring-[hsl(var(--primary))] focus-visible:border-[hsl(var(--primary))] transition-colors"
        />
      </div>
      <div className="flex flex-wrap gap-3 items-center">
        <Filter className="h-4 w-4 text-primary-dark" />
        <FacultySelect
          value={filters.faculty}
          onValueChange={(v) => update("faculty", v)}
          placeholder="Fakultet"
          className="w-[200px] bg-card border-[hsl(var(--primary-dark))] text-primary-dark focus:ring-[hsl(var(--primary))] focus:border-[hsl(var(--primary))]"
        />
        <Select value={filters.year} onValueChange={(v) => update("year", v)}>
          <SelectTrigger className="w-[140px] bg-card border-[hsl(var(--primary-dark))] text-primary-dark focus:ring-[hsl(var(--primary))] focus:border-[hsl(var(--primary))] transition-colors">
            <SelectValue placeholder="Godina" />
          </SelectTrigger>
          <SelectContent>
            {[1, 2, 3, 4, 5, 6].map((y) => (
              <SelectItem key={y} value={String(y)}>{y}. godina</SelectItem>
            ))}
          </SelectContent>
        </Select>
        <Select value={filters.condition} onValueChange={(v) => update("condition", v)}>
          <SelectTrigger className="w-[140px] bg-card border-[hsl(var(--primary-dark))] text-primary-dark focus:ring-[hsl(var(--primary))] focus:border-[hsl(var(--primary))] transition-colors">
            <SelectValue placeholder="Stanje" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="novo">Novo</SelectItem>
            <SelectItem value="kao novo">Kao novo</SelectItem>
            <SelectItem value="dobro">Dobro</SelectItem>
            <SelectItem value="korisceno">Korišćeno</SelectItem>
          </SelectContent>
        </Select>
        {hasFilters && (
          <Button variant="ghost" size="sm" onClick={clearFilters}>
            <X className="h-3 w-3 mr-1" /> Obriši filtere
          </Button>
        )}
      </div>
    </div>
  );
}
import { useState, useEffect } from "react";
import { Star, Loader2, MessageSquare } from "lucide-react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { supabase } from "@/integrations/supabase/client";
import { format } from "date-fns";
import { sr } from "date-fns/locale";

interface Review {
  id: string;
  rating: number;
  comment: string;
  created_at: string;
  reviewer_name: string;
}

interface SellerReviewsProps {
  sellerId: string;
}

export function SellerReviews({ sellerId }: SellerReviewsProps) {
  const [reviews, setReviews] = useState<Review[]>([]);
  const [loading, setLoading] = useState(true);
  const [avgRating, setAvgRating] = useState(0);

  useEffect(() => {
    fetchReviews();
  }, [sellerId]);

  const fetchReviews = async () => {
    setLoading(true);
    const { data } = await supabase
      .from("reviews")
      .select("id, rating, comment, created_at, reviewer_id")
      .eq("seller_id", sellerId)
      .order("created_at", { ascending: false });

    if (!data || data.length === 0) {
      setReviews([]);
      setAvgRating(0);
      setLoading(false);
      return;
    }

    // Fetch reviewer names
    const reviewerIds = [...new Set(data.map((r) => r.reviewer_id))];
    const { data: profiles } = await supabase
      .from("profiles")
      .select("user_id, full_name")
      .in("user_id", reviewerIds);

    const nameMap = new Map(profiles?.map((p) => [p.user_id, p.full_name || "Korisnik"]) || []);

    const mapped: Review[] = data.map((r) => ({
      id: r.id,
      rating: r.rating,
      comment: r.comment,
      created_at: r.created_at,
      reviewer_name: nameMap.get(r.reviewer_id) || "Korisnik",
    }));

    setReviews(mapped);
    const sum = data.reduce((acc, r) => acc + r.rating, 0);
    setAvgRating(sum / data.length);
    setLoading(false);
  };

  if (loading) {
    return (
      <div className="flex justify-center py-8">
        <Loader2 className="h-6 w-6 animate-spin text-primary" />
      </div>
    );
  }

  const renderStars = (rating: number, size = "h-4 w-4") => (
    <div className="flex gap-0.5">
      {[1, 2, 3, 4, 5].map((star) => (
        <Star
          key={star}
          className={`${size} ${
            star <= rating ? "fill-yellow-400 text-yellow-400" : "text-muted-foreground/20"
          }`}
        />
      ))}
    </div>
  );

  return (
    <Card className="border-border bg-card">
      <CardHeader className="pb-3">
        <CardTitle className="flex items-center gap-2 font-display text-lg">
          <MessageSquare className="h-5 w-5 text-primary" />
          Recenzije prodavca
        </CardTitle>
        {reviews.length > 0 && (
          <div className="flex items-center gap-3 pt-1">
            {renderStars(Math.round(avgRating), "h-5 w-5")}
            <span className="text-sm font-medium text-foreground">
              {avgRating.toFixed(1)}
            </span>
            <span className="text-sm text-muted-foreground">
              ({reviews.length} {reviews.length === 1 ? "recenzija" : "recenzija"})
            </span>
          </div>
        )}
      </CardHeader>
      <CardContent>
        {reviews.length === 0 ? (
          <p className="py-4 text-center text-sm text-muted-foreground">
            Ovaj prodavac još nema recenzija.
          </p>
        ) : (
          <div className="space-y-4">
            {reviews.map((review) => (
              <div key={review.id} className="rounded-lg border border-border bg-secondary/30 p-4">
                <div className="flex items-center justify-between">
                  <div className="flex items-center gap-2">
                    {renderStars(review.rating)}
                    <span className="text-sm font-medium text-foreground">{review.reviewer_name}</span>
                  </div>
                  <span className="text-xs text-muted-foreground">
                    {format(new Date(review.created_at), "d. MMM yyyy", { locale: sr })}
                  </span>
                </div>
                <p className="mt-2 text-sm text-muted-foreground">{review.comment}</p>
              </div>
            ))}
          </div>
        )}
      </CardContent>
    </Card>
  );
}
import { useState, useEffect } from "react";
import { Tables } from "@/integrations/supabase/types";
import { supabase } from "@/integrations/supabase/client";
import { BookCard } from "./BookCard";

interface SimilarBooksProps {
  book: Tables<"books">;
}

export function SimilarBooks({ book }: SimilarBooksProps) {
  const [similar, setSimilar] = useState<Tables<"books">[]>([]);

  useEffect(() => {
    const fetch = async () => {
      const { data } = await supabase
        .from("books")
        .select("*")
        .eq("is_sold", false)
        .neq("id", book.id)
        .or(`faculty.eq.${book.faculty},subject.eq.${book.subject}`)
        .order("created_at", { ascending: false })
        .limit(4);
      setSimilar(data || []);
    };
    fetch();
  }, [book.id, book.faculty, book.subject]);

  if (similar.length === 0) return null;

  return (
    <div className="mt-12">
      <h2 className="mb-6 font-display text-xl font-bold text-foreground">Slične knjige</h2>
      <div className="grid gap-5 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
        {similar.map((b) => (
          <BookCard key={b.id} book={b} />
        ))}
      </div>
    </div>
  );
}
import * as React from "react";

const MOBILE_BREAKPOINT = 768;

export function useIsMobile() {
  const [isMobile, setIsMobile] = React.useState<boolean | undefined>(undefined);

  React.useEffect(() => {
    const mql = window.matchMedia(`(max-width: ${MOBILE_BREAKPOINT - 1}px)`);
    const onChange = () => {
      setIsMobile(window.innerWidth < MOBILE_BREAKPOINT);
    };
    mql.addEventListener("change", onChange);
    setIsMobile(window.innerWidth < MOBILE_BREAKPOINT);
    return () => mql.removeEventListener("change", onChange);
  }, []);

  return !!isMobile;
}
import * as React from "react";

import type { ToastActionElement, ToastProps } from "@/components/ui/toast";

const TOAST_LIMIT = 1;
const TOAST_REMOVE_DELAY = 1000000;

type ToasterToast = ToastProps & {
  id: string;
  title?: React.ReactNode;
  description?: React.ReactNode;
  action?: ToastActionElement;
};

const actionTypes = {
  ADD_TOAST: "ADD_TOAST",
  UPDATE_TOAST: "UPDATE_TOAST",
  DISMISS_TOAST: "DISMISS_TOAST",
  REMOVE_TOAST: "REMOVE_TOAST",
} as const;

let count = 0;

function genId() {
  count = (count + 1) % Number.MAX_SAFE_INTEGER;
  return count.toString();
}

type ActionType = typeof actionTypes;

type Action =
  | {
      type: ActionType["ADD_TOAST"];
      toast: ToasterToast;
    }
  | {
      type: ActionType["UPDATE_TOAST"];
      toast: Partial<ToasterToast>;
    }
  | {
      type: ActionType["DISMISS_TOAST"];
      toastId?: ToasterToast["id"];
    }
  | {
      type: ActionType["REMOVE_TOAST"];
      toastId?: ToasterToast["id"];
    };

interface State {
  toasts: ToasterToast[];
}

const toastTimeouts = new Map<string, ReturnType<typeof setTimeout>>();

const addToRemoveQueue = (toastId: string) => {
  if (toastTimeouts.has(toastId)) {
    return;
  }

  const timeout = setTimeout(() => {
    toastTimeouts.delete(toastId);
    dispatch({
      type: "REMOVE_TOAST",
      toastId: toastId,
    });
  }, TOAST_REMOVE_DELAY);

  toastTimeouts.set(toastId, timeout);
};

export const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case "ADD_TOAST":
      return {
        ...state,
        toasts: [action.toast, ...state.toasts].slice(0, TOAST_LIMIT),
      };

    case "UPDATE_TOAST":
      return {
        ...state,
        toasts: state.toasts.map((t) => (t.id === action.toast.id ? { ...t, ...action.toast } : t)),
      };

    case "DISMISS_TOAST": {
      const { toastId } = action;

      // ! Side effects ! - This could be extracted into a dismissToast() action,
      // but I'll keep it here for simplicity
      if (toastId) {
        addToRemoveQueue(toastId);
      } else {
        state.toasts.forEach((toast) => {
          addToRemoveQueue(toast.id);
        });
      }

      return {
        ...state,
        toasts: state.toasts.map((t) =>
          t.id === toastId || toastId === undefined
            ? {
                ...t,
                open: false,
              }
            : t,
        ),
      };
    }
    case "REMOVE_TOAST":
      if (action.toastId === undefined) {
        return {
          ...state,
          toasts: [],
        };
      }
      return {
        ...state,
        toasts: state.toasts.filter((t) => t.id !== action.toastId),
      };
  }
};

const listeners: Array<(state: State) => void> = [];

let memoryState: State = { toasts: [] };

function dispatch(action: Action) {
  memoryState = reducer(memoryState, action);
  listeners.forEach((listener) => {
    listener(memoryState);
  });
}

type Toast = Omit<ToasterToast, "id">;

function toast({ ...props }: Toast) {
  const id = genId();

  const update = (props: ToasterToast) =>
    dispatch({
      type: "UPDATE_TOAST",
      toast: { ...props, id },
    });
  const dismiss = () => dispatch({ type: "DISMISS_TOAST", toastId: id });

  dispatch({
    type: "ADD_TOAST",
    toast: {
      ...props,
      id,
      open: true,
      onOpenChange: (open) => {
        if (!open) dismiss();
      },
    },
  });

  return {
    id: id,
    dismiss,
    update,
  };
}

function useToast() {
  const [state, setState] = React.useState<State>(memoryState);

  React.useEffect(() => {
    listeners.push(setState);
    return () => {
      const index = listeners.indexOf(setState);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    };
  }, [state]);

  return {
    ...state,
    toast,
    dismiss: (toastId?: string) => dispatch({ type: "DISMISS_TOAST", toastId }),
  };
}

export { useToast, toast };
import { useState, useEffect } from "react";
import { supabase } from "@/integrations/supabase/client";
import { useAuth } from "@/lib/auth-context";

export function useAdmin() {
  const { user } = useAuth();
  const [isAdmin, setIsAdmin] = useState(false);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!user) {
      setIsAdmin(false);
      setLoading(false);
      return;
    }
    checkAdmin();
  }, [user]);

  const checkAdmin = async () => {
    setLoading(true);
    const { data } = await supabase
      .from("user_roles")
      .select("role")
      .eq("user_id", user!.id)
      .eq("role", "admin")
      .maybeSingle();
    setIsAdmin(!!data);
    setLoading(false);
  };

  const adminCall = async (action: string, params: Record<string, any> = {}) => {
    const { data: { session } } = await supabase.auth.getSession();
    if (!session) throw new Error("Not authenticated");

    const res = await supabase.functions.invoke("admin", {
      body: { action, ...params },
    });

    if (res.error) throw new Error(res.error.message);
    return res.data;
  };

  return { isAdmin, loading, adminCall };
}
import { useState, useEffect, useCallback } from "react";
import { supabase } from "@/integrations/supabase/client";
import { useAuth } from "@/lib/auth-context";
import { toast } from "sonner";

export function useSavedBooks() {
  const { user } = useAuth();
  const [savedIds, setSavedIds] = useState<Set<string>>(new Set());

  useEffect(() => {
    if (!user) { setSavedIds(new Set()); return; }
    supabase
      .from("saved_books")
      .select("book_id")
      .eq("user_id", user.id)
      .then(({ data }) => {
        setSavedIds(new Set(data?.map((d: any) => d.book_id) || []));
      });
  }, [user]);

  const toggleSave = useCallback(async (bookId: string) => {
    if (!user) {
      toast.error("Morate biti prijavljeni da biste sačuvali knjigu");
      return;
    }
    const isSaved = savedIds.has(bookId);
    if (isSaved) {
      setSavedIds((prev) => { const n = new Set(prev); n.delete(bookId); return n; });
      await supabase.from("saved_books").delete().eq("user_id", user.id).eq("book_id", bookId);
      toast.success("Knjiga uklonjena iz sačuvanih");
    } else {
      setSavedIds((prev) => new Set(prev).add(bookId));
      await supabase.from("saved_books").insert({ user_id: user.id, book_id: bookId });
      toast.success("Knjiga sačuvana");
    }
  }, [user, savedIds]);

  return { savedIds, toggleSave };
}
// This file is automatically generated. Do not edit it directly.
import { createClient } from '@supabase/supabase-js';
import type { Database } from './types';

const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL;
const SUPABASE_PUBLISHABLE_KEY = import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY;

// Import the supabase client like this:
// import { supabase } from "@/integrations/supabase/client";

export const supabase = createClient<Database>(SUPABASE_URL, SUPABASE_PUBLISHABLE_KEY, {
  auth: {
    storage: localStorage,
    persistSession: true,
    autoRefreshToken: true,
  }
});
export type Json =
  | string
  | number
  | boolean
  | null
  | { [key: string]: Json | undefined }
  | Json[]

export type Database = {
  // Allows to automatically instantiate createClient with right options
  // instead of createClient<Database, { PostgrestVersion: 'XX' }>(URL, KEY)
  __InternalSupabase: {
    PostgrestVersion: "14.1"
  }
  public: {
    Tables: {
      books: {
        Row: {
          author: string
          condition: string
          created_at: string
          description: string | null
          faculty: string
          id: string
          image_url: string | null
          is_sold: boolean
          price: number
          seller_id: string
          subject: string
          title: string
          updated_at: string
          year_of_study: number
        }
        Insert: {
          author: string
          condition: string
          created_at?: string
          description?: string | null
          faculty: string
          id?: string
          image_url?: string | null
          is_sold?: boolean
          price: number
          seller_id: string
          subject: string
          title: string
          updated_at?: string
          year_of_study: number
        }
        Update: {
          author?: string
          condition?: string
          created_at?: string
          description?: string | null
          faculty?: string
          id?: string
          image_url?: string | null
          is_sold?: boolean
          price?: number
          seller_id?: string
          subject?: string
          title?: string
          updated_at?: string
          year_of_study?: number
        }
        Relationships: []
      }
      contact_messages: {
        Row: {
          created_at: string
          email: string
          full_name: string
          id: string
          message: string
          subject: string
        }
        Insert: {
          created_at?: string
          email: string
          full_name: string
          id?: string
          message: string
          subject: string
        }
        Update: {
          created_at?: string
          email?: string
          full_name?: string
          id?: string
          message?: string
          subject?: string
        }
        Relationships: []
      }
      conversations: {
        Row: {
          book_id: string
          buyer_id: string
          created_at: string
          id: string
          last_message_at: string
          seller_id: string
        }
        Insert: {
          book_id: string
          buyer_id: string
          created_at?: string
          id?: string
          last_message_at?: string
          seller_id: string
        }
        Update: {
          book_id?: string
          buyer_id?: string
          created_at?: string
          id?: string
          last_message_at?: string
          seller_id?: string
        }
        Relationships: [
          {
            foreignKeyName: "conversations_book_id_fkey"
            columns: ["book_id"]
            isOneToOne: false
            referencedRelation: "books"
            referencedColumns: ["id"]
          },
        ]
      }
      faculties: {
        Row: {
          created_at: string
          id: string
          name: string
        }
        Insert: {
          created_at?: string
          id?: string
          name: string
        }
        Update: {
          created_at?: string
          id?: string
          name?: string
        }
        Relationships: []
      }
      inquiries: {
        Row: {
          book_id: string
          buyer_email: string
          buyer_name: string
          created_at: string
          id: string
          message: string
          seller_id: string
        }
        Insert: {
          book_id: string
          buyer_email: string
          buyer_name: string
          created_at?: string
          id?: string
          message: string
          seller_id: string
        }
        Update: {
          book_id?: string
          buyer_email?: string
          buyer_name?: string
          created_at?: string
          id?: string
          message?: string
          seller_id?: string
        }
        Relationships: [
          {
            foreignKeyName: "inquiries_book_id_fkey"
            columns: ["book_id"]
            isOneToOne: false
            referencedRelation: "books"
            referencedColumns: ["id"]
          },
        ]
      }
      materials: {
        Row: {
          created_at: string
          description: string | null
          faculty: string
          file_name: string
          file_type: string
          file_url: string
          id: string
          subject: string
          title: string
          user_id: string
        }
        Insert: {
          created_at?: string
          description?: string | null
          faculty: string
          file_name: string
          file_type: string
          file_url: string
          id?: string
          subject: string
          title: string
          user_id: string
        }
        Update: {
          created_at?: string
          description?: string | null
          faculty?: string
          file_name?: string
          file_type?: string
          file_url?: string
          id?: string
          subject?: string
          title?: string
          user_id?: string
        }
        Relationships: []
      }
      messages: {
        Row: {
          conversation_id: string
          created_at: string
          id: string
          is_read: boolean
          message_text: string
          sender_id: string
        }
        Insert: {
          conversation_id: string
          created_at?: string
          id?: string
          is_read?: boolean
          message_text: string
          sender_id: string
        }
        Update: {
          conversation_id?: string
          created_at?: string
          id?: string
          is_read?: boolean
          message_text?: string
          sender_id?: string
        }
        Relationships: [
          {
            foreignKeyName: "messages_conversation_id_fkey"
            columns: ["conversation_id"]
            isOneToOne: false
            referencedRelation: "conversations"
            referencedColumns: ["id"]
          },
        ]
      }
      profiles: {
        Row: {
          avatar_url: string | null
          created_at: string
          faculty: string | null
          full_name: string | null
          id: string
          is_suspended: boolean
          last_seen_at: string | null
          role: string
          updated_at: string
          user_id: string
          year_of_study: number | null
        }
        Insert: {
          avatar_url?: string | null
          created_at?: string
          faculty?: string | null
          full_name?: string | null
          id?: string
          is_suspended?: boolean
          last_seen_at?: string | null
          role?: string
          updated_at?: string
          user_id: string
          year_of_study?: number | null
        }
        Update: {
          avatar_url?: string | null
          created_at?: string
          faculty?: string | null
          full_name?: string | null
          id?: string
          is_suspended?: boolean
          last_seen_at?: string | null
          role?: string
          updated_at?: string
          user_id?: string
          year_of_study?: number | null
        }
        Relationships: []
      }
      reports: {
        Row: {
          book_id: string
          created_at: string
          description: string | null
          id: string
          reason: string
          resolved: boolean
          user_id: string
        }
        Insert: {
          book_id: string
          created_at?: string
          description?: string | null
          id?: string
          reason: string
          resolved?: boolean
          user_id: string
        }
        Update: {
          book_id?: string
          created_at?: string
          description?: string | null
          id?: string
          reason?: string
          resolved?: boolean
          user_id?: string
        }
        Relationships: [
          {
            foreignKeyName: "reports_book_id_fkey"
            columns: ["book_id"]
            isOneToOne: false
            referencedRelation: "books"
            referencedColumns: ["id"]
          },
        ]
      }
      reviews: {
        Row: {
          book_id: string
          comment: string
          conversation_id: string
          created_at: string
          id: string
          rating: number
          reviewer_id: string
          seller_id: string
        }
        Insert: {
          book_id: string
          comment?: string
          conversation_id: string
          created_at?: string
          id?: string
          rating: number
          reviewer_id: string
          seller_id: string
        }
        Update: {
          book_id?: string
          comment?: string
          conversation_id?: string
          created_at?: string
          id?: string
          rating?: number
          reviewer_id?: string
          seller_id?: string
        }
        Relationships: [
          {
            foreignKeyName: "reviews_book_id_fkey"
            columns: ["book_id"]
            isOneToOne: false
            referencedRelation: "books"
            referencedColumns: ["id"]
          },
          {
            foreignKeyName: "reviews_conversation_id_fkey"
            columns: ["conversation_id"]
            isOneToOne: false
            referencedRelation: "conversations"
            referencedColumns: ["id"]
          },
        ]
      }
      saved_books: {
        Row: {
          book_id: string
          created_at: string
          id: string
          user_id: string
        }
        Insert: {
          book_id: string
          created_at?: string
          id?: string
          user_id: string
        }
        Update: {
          book_id?: string
          created_at?: string
          id?: string
          user_id?: string
        }
        Relationships: [
          {
            foreignKeyName: "saved_books_book_id_fkey"
            columns: ["book_id"]
            isOneToOne: false
            referencedRelation: "books"
            referencedColumns: ["id"]
          },
        ]
      }
      support_conversations: {
        Row: {
          created_at: string
          id: string
          last_message_at: string
          subject: string
          user_id: string
        }
        Insert: {
          created_at?: string
          id?: string
          last_message_at?: string
          subject: string
          user_id: string
        }
        Update: {
          created_at?: string
          id?: string
          last_message_at?: string
          subject?: string
          user_id?: string
        }
        Relationships: []
      }
      support_messages: {
        Row: {
          conversation_id: string
          created_at: string
          id: string
          is_admin: boolean
          is_read: boolean
          message_text: string
          sender_id: string
        }
        Insert: {
          conversation_id: string
          created_at?: string
          id?: string
          is_admin?: boolean
          is_read?: boolean
          message_text: string
          sender_id: string
        }
        Update: {
          conversation_id?: string
          created_at?: string
          id?: string
          is_admin?: boolean
          is_read?: boolean
          message_text?: string
          sender_id?: string
        }
        Relationships: [
          {
            foreignKeyName: "support_messages_conversation_id_fkey"
            columns: ["conversation_id"]
            isOneToOne: false
            referencedRelation: "support_conversations"
            referencedColumns: ["id"]
          },
        ]
      }
      user_roles: {
        Row: {
          id: string
          role: Database["public"]["Enums"]["app_role"]
          user_id: string
        }
        Insert: {
          id?: string
          role: Database["public"]["Enums"]["app_role"]
          user_id: string
        }
        Update: {
          id?: string
          role?: Database["public"]["Enums"]["app_role"]
          user_id?: string
        }
        Relationships: []
      }
    }
    Views: {
      [_ in never]: never
    }
    Functions: {
      can_review_conversation: {
        Args: { _conversation_id: string; _user_id: string }
        Returns: boolean
      }
      get_conversation_seller: {
        Args: { _conversation_id: string }
        Returns: string
      }
      has_role: {
        Args: {
          _role: Database["public"]["Enums"]["app_role"]
          _user_id: string
        }
        Returns: boolean
      }
      is_conversation_participant: {
        Args: { _conversation_id: string; _user_id: string }
        Returns: boolean
      }
      is_support_participant: {
        Args: { _conversation_id: string; _user_id: string }
        Returns: boolean
      }
    }
    Enums: {
      app_role: "admin" | "moderator" | "user"
    }
    CompositeTypes: {
      [_ in never]: never
    }
  }
}

type DatabaseWithoutInternals = Omit<Database, "__InternalSupabase">

type DefaultSchema = DatabaseWithoutInternals[Extract<keyof Database, "public">]

export type Tables<
  DefaultSchemaTableNameOrOptions extends
    | keyof (DefaultSchema["Tables"] & DefaultSchema["Views"])
    | { schema: keyof DatabaseWithoutInternals },
  TableName extends DefaultSchemaTableNameOrOptions extends {
    schema: keyof DatabaseWithoutInternals
  }
    ? keyof (DatabaseWithoutInternals[DefaultSchemaTableNameOrOptions["schema"]]["Tables"] &
        DatabaseWithoutInternals[DefaultSchemaTableNameOrOptions["schema"]]["Views"])
    : never = never,
> = DefaultSchemaTableNameOrOptions extends {
  schema: keyof DatabaseWithoutInternals
}
  ? (DatabaseWithoutInternals[DefaultSchemaTableNameOrOptions["schema"]]["Tables"] &
      DatabaseWithoutInternals[DefaultSchemaTableNameOrOptions["schema"]]["Views"])[TableName] extends {
      Row: infer R
    }
    ? R
    : never
  : DefaultSchemaTableNameOrOptions extends keyof (DefaultSchema["Tables"] &
        DefaultSchema["Views"])
    ? (DefaultSchema["Tables"] &
        DefaultSchema["Views"])[DefaultSchemaTableNameOrOptions] extends {
        Row: infer R
      }
      ? R
      : never
    : never

export type TablesInsert<
  DefaultSchemaTableNameOrOptions extends
    | keyof DefaultSchema["Tables"]
    | { schema: keyof DatabaseWithoutInternals },
  TableName extends DefaultSchemaTableNameOrOptions extends {
    schema: keyof DatabaseWithoutInternals
  }
    ? keyof DatabaseWithoutInternals[DefaultSchemaTableNameOrOptions["schema"]]["Tables"]
    : never = never,
> = DefaultSchemaTableNameOrOptions extends {
  schema: keyof DatabaseWithoutInternals
}
  ? DatabaseWithoutInternals[DefaultSchemaTableNameOrOptions["schema"]]["Tables"][TableName] extends {
      Insert: infer I
    }
    ? I
    : never
  : DefaultSchemaTableNameOrOptions extends keyof DefaultSchema["Tables"]
    ? DefaultSchema["Tables"][DefaultSchemaTableNameOrOptions] extends {
        Insert: infer I
      }
      ? I
      : never
    : never

export type TablesUpdate<
  DefaultSchemaTableNameOrOptions extends
    | keyof DefaultSchema["Tables"]
    | { schema: keyof DatabaseWithoutInternals },
  TableName extends DefaultSchemaTableNameOrOptions extends {
    schema: keyof DatabaseWithoutInternals
  }
    ? keyof DatabaseWithoutInternals[DefaultSchemaTableNameOrOptions["schema"]]["Tables"]
    : never = never,
> = DefaultSchemaTableNameOrOptions extends {
  schema: keyof DatabaseWithoutInternals
}
  ? DatabaseWithoutInternals[DefaultSchemaTableNameOrOptions["schema"]]["Tables"][TableName] extends {
      Update: infer U
    }
    ? U
    : never
  : DefaultSchemaTableNameOrOptions extends keyof DefaultSchema["Tables"]
    ? DefaultSchema["Tables"][DefaultSchemaTableNameOrOptions] extends {
        Update: infer U
      }
      ? U
      : never
    : never

export type Enums<
  DefaultSchemaEnumNameOrOptions extends
    | keyof DefaultSchema["Enums"]
    | { schema: keyof DatabaseWithoutInternals },
  EnumName extends DefaultSchemaEnumNameOrOptions extends {
    schema: keyof DatabaseWithoutInternals
  }
    ? keyof DatabaseWithoutInternals[DefaultSchemaEnumNameOrOptions["schema"]]["Enums"]
    : never = never,
> = DefaultSchemaEnumNameOrOptions extends {
  schema: keyof DatabaseWithoutInternals
}
  ? DatabaseWithoutInternals[DefaultSchemaEnumNameOrOptions["schema"]]["Enums"][EnumName]
  : DefaultSchemaEnumNameOrOptions extends keyof DefaultSchema["Enums"]
    ? DefaultSchema["Enums"][DefaultSchemaEnumNameOrOptions]
    : never

export type CompositeTypes<
  PublicCompositeTypeNameOrOptions extends
    | keyof DefaultSchema["CompositeTypes"]
    | { schema: keyof DatabaseWithoutInternals },
  CompositeTypeName extends PublicCompositeTypeNameOrOptions extends {
    schema: keyof DatabaseWithoutInternals
  }
    ? keyof DatabaseWithoutInternals[PublicCompositeTypeNameOrOptions["schema"]]["CompositeTypes"]
    : never = never,
> = PublicCompositeTypeNameOrOptions extends {
  schema: keyof DatabaseWithoutInternals
}
  ? DatabaseWithoutInternals[PublicCompositeTypeNameOrOptions["schema"]]["CompositeTypes"][CompositeTypeName]
  : PublicCompositeTypeNameOrOptions extends keyof DefaultSchema["CompositeTypes"]
    ? DefaultSchema["CompositeTypes"][PublicCompositeTypeNameOrOptions]
    : never

export const Constants = {
  public: {
    Enums: {
      app_role: ["admin", "moderator", "user"],
    },
  },
} as const
import { createContext, useContext, useEffect, useState, useRef, ReactNode } from "react";
import { User, Session } from "@supabase/supabase-js";
import { supabase } from "@/integrations/supabase/client";

interface AuthContextType {
  user: User | null;
  session: Session | null;
  loading: boolean;
  signUp: (email: string, password: string, fullName: string) => Promise<{ error: Error | null }>;
  signIn: (email: string, password: string) => Promise<{ error: Error | null }>;
  signOut: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [session, setSession] = useState<Session | null>(null);
  const [loading, setLoading] = useState(true);
  const intervalRef = useRef<ReturnType<typeof setInterval> | null>(null);

  useEffect(() => {
    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      setSession(session);
      setUser(session?.user ?? null);
      setLoading(false);
    });

    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session);
      setUser(session?.user ?? null);
      setLoading(false);
    });

    return () => subscription.unsubscribe();
  }, []);

  // Update last_seen_at periodically
  useEffect(() => {
    if (!user) {
      if (intervalRef.current) clearInterval(intervalRef.current);
      return;
    }
    const updateLastSeen = () => {
      supabase.from("profiles").update({ last_seen_at: new Date().toISOString() }).eq("user_id", user.id).then();
    };
    updateLastSeen();
    intervalRef.current = setInterval(updateLastSeen, 2 * 60 * 1000); // every 2 min
    return () => { if (intervalRef.current) clearInterval(intervalRef.current); };
  }, [user]);

  const signUp = async (email: string, password: string, fullName: string) => {
    const { error } = await supabase.auth.signUp({
      email,
      password,
      options: {
        data: { full_name: fullName },
        emailRedirectTo: window.location.origin,
      },
    });
    return { error: error ? new Error(error.message) : null };
  };

  const signIn = async (email: string, password: string) => {
    const { error } = await supabase.auth.signInWithPassword({ email, password });
    return { error: error ? new Error(error.message) : null };
  };

  const signOut = async () => {
    await supabase.auth.signOut();
  };

  return (
    <AuthContext.Provider value={{ user, session, loading, signUp, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error("useAuth must be used within AuthProvider");
  return context;
}
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
import { useState, useEffect } from "react";
import { BookOpen, Flag, Users, Loader2 } from "lucide-react";
import { Card, CardContent } from "@/components/ui/card";
import { useAdmin } from "@/hooks/useAdmin";

export default function AdminDashboard() {
  const { adminCall } = useAdmin();
  const [stats, setStats] = useState({ total_listings: 0, open_reports: 0, total_users: 0 });
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetch = async () => {
      try {
        const data = await adminCall("get_stats");
        setStats(data.stats);
      } catch (e) {
        console.error(e);
      }
      setLoading(false);
    };
    fetch();
  }, []);

  if (loading) {
    return (
      <div className="flex justify-center py-20">
        <Loader2 className="h-8 w-8 animate-spin text-primary" />
      </div>
    );
  }

  const cards = [
    { label: "Ukupno oglasa", value: stats.total_listings, icon: BookOpen, color: "text-primary" },
    { label: "Otvorene prijave", value: stats.open_reports, icon: Flag, color: "text-destructive" },
    { label: "Ukupno korisnika", value: stats.total_users, icon: Users, color: "text-primary" },
  ];

  return (
    <div className="grid gap-4 sm:grid-cols-3">
      {cards.map((c) => (
        <Card key={c.label}>
          <CardContent className="flex items-center gap-4 p-6">
            <div className="flex h-12 w-12 items-center justify-center rounded-xl bg-secondary">
              <c.icon className={`h-6 w-6 ${c.color}`} />
            </div>
            <div>
              <p className="font-display text-2xl font-bold text-foreground">{c.value}</p>
              <p className="text-sm text-muted-foreground">{c.label}</p>
            </div>
          </CardContent>
        </Card>
      ))}
    </div>
  );
}
import { useEffect } from "react";
import { Outlet, useNavigate, Link, useLocation } from "react-router-dom";
import { useAuth } from "@/lib/auth-context";
import { useAdmin } from "@/hooks/useAdmin";
import { Navbar } from "@/components/Navbar";
import { Loader2, LayoutDashboard, BookOpen, Flag, Users, MessageSquare } from "lucide-react";
import { Button } from "@/components/ui/button";

const NAV_ITEMS = [
  { to: "/admin", icon: LayoutDashboard, label: "Pregled", exact: true },
  { to: "/admin/listings", icon: BookOpen, label: "Oglasi" },
  { to: "/admin/reports", icon: Flag, label: "Prijave" },
  { to: "/admin/users", icon: Users, label: "Korisnici" },
  { to: "/admin/support", icon: MessageSquare, label: "Podrška" },
];

export default function AdminLayout() {
  const { user, loading: authLoading } = useAuth();
  const { isAdmin, loading: adminLoading } = useAdmin();
  const navigate = useNavigate();
  const location = useLocation();

  useEffect(() => {
    if (!authLoading && !user) navigate("/auth");
    if (!authLoading && !adminLoading && user && !isAdmin) navigate("/");
  }, [user, authLoading, isAdmin, adminLoading]);

  if (authLoading || adminLoading) {
    return (
      <div className="min-h-screen bg-background">
        <Navbar />
        <div className="flex justify-center py-32">
          <Loader2 className="h-10 w-10 animate-spin text-primary" />
        </div>
      </div>
    );
  }

  if (!isAdmin) return null;

  return (
    <div className="min-h-screen bg-background overflow-x-hidden">
      <Navbar />
      <div className="container mx-auto px-4 py-6 max-w-7xl">
        <div className="mb-6 flex items-center gap-3">
          <div className="flex h-10 w-10 shrink-0 items-center justify-center rounded-xl gradient-primary">
            <LayoutDashboard className="h-5 w-5 text-primary-foreground" />
          </div>
          <h1 className="font-display text-2xl font-bold text-foreground">Admin Panel</h1>
        </div>

        {/* Nav tabs */}
        <div className="mb-6 flex gap-1 overflow-x-auto rounded-lg border border-border bg-card p-1">
          {NAV_ITEMS.map((item) => {
            const isActive = item.exact
              ? location.pathname === item.to
              : location.pathname.startsWith(item.to);
            return (
              <Link key={item.to} to={item.to}>
                <Button
                  variant={isActive ? "default" : "ghost"}
                  size="sm"
                  className={isActive ? "gradient-primary border-0" : ""}
                >
                  <item.icon className="mr-1.5 h-4 w-4" />
                  {item.label}
                </Button>
              </Link>
            );
          })}
        </div>

        <div className="min-h-[60vh]">
          <Outlet />
        </div>
      </div>
    </div>
  );
}
import { useState, useEffect } from "react";
import { Loader2, Trash2, Eye, CheckCircle, XCircle } from "lucide-react";
import { supabase } from "@/integrations/supabase/client";
import { useAdmin } from "@/hooks/useAdmin";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";
import {
  AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent,
  AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger,
} from "@/components/ui/alert-dialog";
import { toast } from "sonner";
import { useNavigate } from "react-router-dom";

interface BookRow {
  id: string;
  title: string;
  faculty: string;
  price: number;
  is_sold: boolean;
  created_at: string;
  seller_name: string;
}

export default function AdminListings() {
  const { adminCall } = useAdmin();
  const navigate = useNavigate();
  const [books, setBooks] = useState<BookRow[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => { fetchBooks(); }, []);

  const fetchBooks = async () => {
    setLoading(true);
    const { data } = await supabase
      .from("books")
      .select("id, title, faculty, price, is_sold, created_at, seller_id")
      .order("created_at", { ascending: false });

    if (!data) { setLoading(false); return; }

    const sellerIds = [...new Set(data.map((b: any) => b.seller_id))];
    const { data: profiles } = await supabase
      .from("profiles")
      .select("user_id, full_name")
      .in("user_id", sellerIds);

    const nameMap: Record<string, string> = {};
    profiles?.forEach((p: any) => { nameMap[p.user_id] = p.full_name || "Nepoznat"; });

    setBooks(data.map((b: any) => ({
      id: b.id,
      title: b.title,
      faculty: b.faculty,
      price: b.price,
      is_sold: b.is_sold,
      created_at: b.created_at,
      seller_name: nameMap[b.seller_id] || "Nepoznat",
    })));
    setLoading(false);
  };

  const handleDelete = async (id: string) => {
    try {
      await adminCall("delete_book", { book_id: id });
      setBooks((prev) => prev.filter((b) => b.id !== id));
      toast.success("Oglas obrisan");
    } catch { toast.error("Greška"); }
  };

  const handleStatusChange = async (id: string, is_sold: boolean) => {
    try {
      await adminCall("update_book_status", { book_id: id, is_sold });
      setBooks((prev) => prev.map((b) => b.id === id ? { ...b, is_sold } : b));
      toast.success("Status ažuriran");
    } catch { toast.error("Greška"); }
  };

  if (loading) {
    return <div className="flex justify-center py-20"><Loader2 className="h-8 w-8 animate-spin text-primary" /></div>;
  }

  return (
    <div className="rounded-lg border border-border bg-card overflow-hidden">
      <div className="overflow-x-auto">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Naslov</TableHead>
              <TableHead className="hidden sm:table-cell">Prodavac</TableHead>
              <TableHead className="hidden md:table-cell">Fakultet</TableHead>
              <TableHead>Cena</TableHead>
              <TableHead>Status</TableHead>
              <TableHead className="hidden md:table-cell">Datum</TableHead>
              <TableHead className="text-right">Akcije</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {books.map((book) => (
              <TableRow key={book.id}>
                <TableCell className="font-medium max-w-[200px] truncate">{book.title}</TableCell>
                <TableCell className="hidden sm:table-cell">{book.seller_name}</TableCell>
                <TableCell className="hidden md:table-cell text-sm">{book.faculty}</TableCell>
                <TableCell>{book.price} RSD</TableCell>
                <TableCell>
                  <Badge variant={book.is_sold ? "destructive" : "secondary"}>
                    {book.is_sold ? "Prodato" : "Aktivno"}
                  </Badge>
                </TableCell>
                <TableCell className="hidden md:table-cell text-sm text-muted-foreground">
                  {new Date(book.created_at).toLocaleDateString("sr-RS")}
                </TableCell>
                <TableCell className="text-right">
                  <div className="flex items-center justify-end gap-1">
                    <Button variant="ghost" size="icon" className="h-8 w-8" onClick={() => navigate(`/listing/${book.id}`)}>
                      <Eye className="h-4 w-4" />
                    </Button>
                    {book.is_sold ? (
                      <Button variant="ghost" size="icon" className="h-8 w-8 text-green-600" onClick={() => handleStatusChange(book.id, false)} title="Označi kao aktivno">
                        <CheckCircle className="h-4 w-4" />
                      </Button>
                    ) : (
                      <Button variant="ghost" size="icon" className="h-8 w-8 text-orange-500" onClick={() => handleStatusChange(book.id, true)} title="Označi kao prodato">
                        <XCircle className="h-4 w-4" />
                      </Button>
                    )}
                    <AlertDialog>
                      <AlertDialogTrigger asChild>
                        <Button variant="ghost" size="icon" className="h-8 w-8 text-destructive">
                          <Trash2 className="h-4 w-4" />
                        </Button>
                      </AlertDialogTrigger>
                      <AlertDialogContent>
                        <AlertDialogHeader>
                          <AlertDialogTitle>Obrisati oglas?</AlertDialogTitle>
                          <AlertDialogDescription>Ova akcija se ne može poništiti. Oglas "{book.title}" će biti trajno obrisan.</AlertDialogDescription>
                        </AlertDialogHeader>
                        <AlertDialogFooter>
                          <AlertDialogCancel>Otkaži</AlertDialogCancel>
                          <AlertDialogAction onClick={() => handleDelete(book.id)}>Obriši</AlertDialogAction>
                        </AlertDialogFooter>
                      </AlertDialogContent>
                    </AlertDialog>
                  </div>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
import { useEffect, useState } from "react";
import { supabase } from "@/integrations/supabase/client";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";
import { Badge } from "@/components/ui/badge";
import { Loader2, Mail } from "lucide-react";
import { format } from "date-fns";

interface ContactMessage {
  id: string;
  full_name: string;
  email: string;
  subject: string;
  message: string;
  created_at: string;
}

export default function AdminMessages() {
  const [messages, setMessages] = useState<ContactMessage[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchMessages();
  }, []);

  const fetchMessages = async () => {
    const { data } = await supabase
      .from("contact_messages")
      .select("*")
      .order("created_at", { ascending: false });
    setMessages((data as ContactMessage[]) || []);
    setLoading(false);
  };

  if (loading) {
    return <div className="flex justify-center py-16"><Loader2 className="h-8 w-8 animate-spin text-primary" /></div>;
  }

  return (
    <Card className="border-border">
      <CardHeader>
        <CardTitle className="flex items-center gap-2 text-lg">
          <Mail className="h-5 w-5 text-primary" />
          Kontakt poruke
          <Badge variant="secondary" className="ml-auto">{messages.length}</Badge>
        </CardTitle>
      </CardHeader>
      <CardContent>
        {messages.length === 0 ? (
          <p className="text-center text-muted-foreground py-8">Nema kontakt poruka.</p>
        ) : (
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Ime</TableHead>
                <TableHead>Email</TableHead>
                <TableHead>Predmet</TableHead>
                <TableHead className="hidden md:table-cell">Poruka</TableHead>
                <TableHead>Datum</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {messages.map((m) => (
                <TableRow key={m.id}>
                  <TableCell className="font-medium">{m.full_name}</TableCell>
                  <TableCell>{m.email}</TableCell>
                  <TableCell>{m.subject}</TableCell>
                  <TableCell className="hidden md:table-cell max-w-xs truncate">{m.message}</TableCell>
                  <TableCell className="whitespace-nowrap">{format(new Date(m.created_at), "dd.MM.yyyy HH:mm")}</TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        )}
      </CardContent>
    </Card>
  );
}
import { useState, useEffect } from "react";
import { Loader2, Eye, CheckCircle, Trash2 } from "lucide-react";
import { supabase } from "@/integrations/supabase/client";
import { useAdmin } from "@/hooks/useAdmin";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";
import { toast } from "sonner";
import { useNavigate } from "react-router-dom";

const REASON_LABELS: Record<string, string> = {
  spam: "Spam",
  incorrect: "Netačan oglas",
  already_sold: "Već prodato",
  inappropriate: "Neprikladni sadržaj",
  other: "Ostalo",
};

interface ReportRow {
  id: string;
  reason: string;
  description: string | null;
  resolved: boolean;
  created_at: string;
  book_id: string;
  book_title: string;
  seller_name: string;
  reporter_name: string;
}

export default function AdminReports() {
  const { adminCall } = useAdmin();
  const navigate = useNavigate();
  const [reports, setReports] = useState<ReportRow[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => { fetchReports(); }, []);

  const fetchReports = async () => {
    setLoading(true);
    const { data } = await supabase
      .from("reports")
      .select("id, reason, description, resolved, created_at, book_id, user_id, books(title, seller_id)")
      .order("created_at", { ascending: false });

    if (!data) { setLoading(false); return; }

    // Get reporter and seller names
    const userIds = new Set<string>();
    data.forEach((r: any) => {
      userIds.add(r.user_id);
      if ((r.books as any)?.seller_id) userIds.add((r.books as any).seller_id);
    });

    const { data: profiles } = await supabase
      .from("profiles")
      .select("user_id, full_name")
      .in("user_id", [...userIds]);

    const nameMap: Record<string, string> = {};
    profiles?.forEach((p: any) => { nameMap[p.user_id] = p.full_name || "Nepoznat"; });

    setReports(data.map((r: any) => ({
      id: r.id,
      reason: r.reason,
      description: r.description,
      resolved: r.resolved,
      created_at: r.created_at,
      book_id: r.book_id,
      book_title: (r.books as any)?.title || "Obrisano",
      seller_name: nameMap[(r.books as any)?.seller_id] || "Nepoznat",
      reporter_name: nameMap[r.user_id] || "Nepoznat",
    })));
    setLoading(false);
  };

  const handleResolve = async (id: string) => {
    try {
      await adminCall("resolve_report", { report_id: id });
      setReports((prev) => prev.map((r) => r.id === id ? { ...r, resolved: true } : r));
      toast.success("Prijava rešena");
    } catch { toast.error("Greška"); }
  };

  const handleDeleteListing = async (report: ReportRow) => {
    try {
      await adminCall("delete_book", { book_id: report.book_id });
      await adminCall("resolve_report", { report_id: report.id });
      setReports((prev) => prev.map((r) => r.id === report.id ? { ...r, resolved: true, book_title: "Obrisano" } : r));
      toast.success("Oglas obrisan i prijava rešena");
    } catch { toast.error("Greška"); }
  };

  if (loading) {
    return <div className="flex justify-center py-20"><Loader2 className="h-8 w-8 animate-spin text-primary" /></div>;
  }

  if (reports.length === 0) {
    return (
      <div className="py-20 text-center">
        <CheckCircle className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
        <h3 className="font-display text-lg font-semibold text-foreground">Nema prijava</h3>
        <p className="text-sm text-muted-foreground">Trenutno nema prijavljenih oglasa.</p>
      </div>
    );
  }

  return (
    <div className="rounded-lg border border-border bg-card overflow-hidden">
      <div className="overflow-x-auto">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Oglas</TableHead>
              <TableHead className="hidden sm:table-cell">Prodavac</TableHead>
              <TableHead className="hidden md:table-cell">Prijavio</TableHead>
              <TableHead>Razlog</TableHead>
              <TableHead>Status</TableHead>
              <TableHead className="hidden md:table-cell">Datum</TableHead>
              <TableHead className="text-right">Akcije</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {reports.map((report) => (
              <TableRow key={report.id}>
                <TableCell className="font-medium max-w-[160px] truncate">{report.book_title}</TableCell>
                <TableCell className="hidden sm:table-cell">{report.seller_name}</TableCell>
                <TableCell className="hidden md:table-cell">{report.reporter_name}</TableCell>
                <TableCell>
                  <span className="text-sm">{REASON_LABELS[report.reason] || report.reason}</span>
                  {report.description && (
                    <p className="text-xs text-muted-foreground mt-0.5 max-w-[200px] truncate">{report.description}</p>
                  )}
                </TableCell>
                <TableCell>
                  <Badge variant={report.resolved ? "secondary" : "destructive"}>
                    {report.resolved ? "Rešeno" : "Otvoreno"}
                  </Badge>
                </TableCell>
                <TableCell className="hidden md:table-cell text-sm text-muted-foreground">
                  {new Date(report.created_at).toLocaleDateString("sr-RS")}
                </TableCell>
                <TableCell className="text-right">
                  <div className="flex items-center justify-end gap-1">
                    <Button variant="ghost" size="icon" className="h-8 w-8" onClick={() => navigate(`/listing/${report.book_id}`)}>
                      <Eye className="h-4 w-4" />
                    </Button>
                    {!report.resolved && (
                      <>
                        <Button variant="ghost" size="icon" className="h-8 w-8 text-green-600" onClick={() => handleResolve(report.id)} title="Označi kao rešeno">
                          <CheckCircle className="h-4 w-4" />
                        </Button>
                        <Button variant="ghost" size="icon" className="h-8 w-8 text-destructive" onClick={() => handleDeleteListing(report)} title="Obriši oglas">
                          <Trash2 className="h-4 w-4" />
                        </Button>
                      </>
                    )}
                  </div>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
import { useEffect, useState } from "react";
import { useNavigate } from "react-router-dom";
import { supabase } from "@/integrations/supabase/client";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Loader2, MessageSquare, ChevronRight } from "lucide-react";
import { format } from "date-fns";

interface SupportConvo {
  id: string;
  user_id: string;
  subject: string;
  created_at: string;
  last_message_at: string;
}

interface Profile {
  user_id: string;
  full_name: string | null;
}

export default function AdminSupport() {
  const navigate = useNavigate();
  const [convos, setConvos] = useState<SupportConvo[]>([]);
  const [profiles, setProfiles] = useState<Record<string, string>>({});
  const [unreadMap, setUnreadMap] = useState<Record<string, number>>({});
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchAll();
  }, []);

  const fetchAll = async () => {
    const { data } = await supabase
      .from("support_conversations")
      .select("*")
      .order("last_message_at", { ascending: false });
    const list = (data as SupportConvo[]) || [];
    setConvos(list);

    // Fetch profiles for user names
    if (list.length > 0) {
      const userIds = [...new Set(list.map((c) => c.user_id))];
      const { data: profs } = await supabase
        .from("profiles")
        .select("user_id, full_name")
        .in("user_id", userIds);
      const map: Record<string, string> = {};
      (profs || []).forEach((p: any) => {
        map[p.user_id] = p.full_name || "Korisnik";
      });
      setProfiles(map);

      // Unread counts (messages from users, not admin)
      const { data: msgs } = await supabase
        .from("support_messages")
        .select("conversation_id")
        .in("conversation_id", list.map((c) => c.id))
        .eq("is_read", false)
        .eq("is_admin", false);
      const umap: Record<string, number> = {};
      (msgs || []).forEach((m: any) => {
        umap[m.conversation_id] = (umap[m.conversation_id] || 0) + 1;
      });
      setUnreadMap(umap);
    }
    setLoading(false);
  };

  if (loading) {
    return <div className="flex justify-center py-16"><Loader2 className="h-8 w-8 animate-spin text-primary" /></div>;
  }

  return (
    <Card className="border-border">
      <CardHeader>
        <CardTitle className="flex items-center gap-2 text-lg">
          <MessageSquare className="h-5 w-5 text-primary" />
          Podrška
          <Badge variant="secondary" className="ml-auto">{convos.length}</Badge>
        </CardTitle>
      </CardHeader>
      <CardContent>
        {convos.length === 0 ? (
          <p className="text-center text-muted-foreground py-8">Nema upita korisnika.</p>
        ) : (
          <div className="space-y-2">
            {convos.map((c) => (
              <div
                key={c.id}
                className="flex items-center justify-between rounded-lg border border-border p-3 cursor-pointer hover:bg-muted/50 transition-colors"
                onClick={() => navigate(`/support/${c.id}`)}
              >
                <div className="flex-1 min-w-0">
                  <div className="flex items-center gap-2">
                    <h4 className="font-medium text-foreground truncate">{c.subject}</h4>
                    {unreadMap[c.id] && (
                      <Badge>{unreadMap[c.id]}</Badge>
                    )}
                  </div>
                  <p className="text-xs text-muted-foreground">
                    {profiles[c.user_id] || "Korisnik"} • {format(new Date(c.last_message_at), "dd.MM.yyyy HH:mm")}
                  </p>
                </div>
                <ChevronRight className="h-5 w-5 text-muted-foreground shrink-0" />
              </div>
            ))}
          </div>
        )}
      </CardContent>
    </Card>
  );
}
import { useState, useEffect } from "react";
import { Loader2, UserX, UserCheck, Users } from "lucide-react";
import { useAdmin } from "@/hooks/useAdmin";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";
import { toast } from "sonner";

interface UserRow {
  id: string;
  email: string;
  full_name: string;
  faculty: string;
  is_suspended: boolean;
  created_at: string;
}

export default function AdminUsers() {
  const { adminCall } = useAdmin();
  const [users, setUsers] = useState<UserRow[]>([]);
  const [loading, setLoading] = useState(true);
  const [actionId, setActionId] = useState<string | null>(null);

  useEffect(() => { fetchUsers(); }, []);

  const fetchUsers = async () => {
    setLoading(true);
    try {
      const data = await adminCall("get_users");
      setUsers(data.users || []);
    } catch (e) {
      console.error(e);
    }
    setLoading(false);
  };

  const handleSuspend = async (userId: string) => {
    setActionId(userId);
    try {
      await adminCall("suspend_user", { user_id: userId });
      setUsers((prev) => prev.map((u) => u.id === userId ? { ...u, is_suspended: true } : u));
      toast.success("Korisnik suspendovan");
    } catch { toast.error("Greška"); }
    setActionId(null);
  };

  const handleReactivate = async (userId: string) => {
    setActionId(userId);
    try {
      await adminCall("reactivate_user", { user_id: userId });
      setUsers((prev) => prev.map((u) => u.id === userId ? { ...u, is_suspended: false } : u));
      toast.success("Korisnik reaktiviran");
    } catch { toast.error("Greška"); }
    setActionId(null);
  };

  if (loading) {
    return <div className="flex justify-center py-20"><Loader2 className="h-8 w-8 animate-spin text-primary" /></div>;
  }

  if (users.length === 0) {
    return (
      <div className="py-20 text-center">
        <Users className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
        <h3 className="font-display text-lg font-semibold text-foreground">Nema korisnika</h3>
      </div>
    );
  }

  return (
    <div className="rounded-lg border border-border bg-card overflow-hidden">
      <div className="overflow-x-auto">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Ime</TableHead>
              <TableHead>Email</TableHead>
              <TableHead className="hidden md:table-cell">Fakultet</TableHead>
              <TableHead>Status</TableHead>
              <TableHead className="hidden md:table-cell">Registracija</TableHead>
              <TableHead className="text-right">Akcije</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {users.map((u) => (
              <TableRow key={u.id}>
                <TableCell className="font-medium">{u.full_name || "—"}</TableCell>
                <TableCell className="text-sm">{u.email}</TableCell>
                <TableCell className="hidden md:table-cell text-sm">{u.faculty || "—"}</TableCell>
                <TableCell>
                  <Badge variant={u.is_suspended ? "destructive" : "secondary"}>
                    {u.is_suspended ? "Suspendovan" : "Aktivan"}
                  </Badge>
                </TableCell>
                <TableCell className="hidden md:table-cell text-sm text-muted-foreground">
                  {new Date(u.created_at).toLocaleDateString("sr-RS")}
                </TableCell>
                <TableCell className="text-right">
                  {u.is_suspended ? (
                    <Button
                      variant="ghost"
                      size="sm"
                      className="text-green-600"
                      disabled={actionId === u.id}
                      onClick={() => handleReactivate(u.id)}
                    >
                      {actionId === u.id ? <Loader2 className="h-4 w-4 animate-spin" /> : <UserCheck className="h-4 w-4 mr-1" />}
                      Reaktiviraj
                    </Button>
                  ) : (
                    <Button
                      variant="ghost"
                      size="sm"
                      className="text-destructive"
                      disabled={actionId === u.id}
                      onClick={() => handleSuspend(u.id)}
                    >
                      {actionId === u.id ? <Loader2 className="h-4 w-4 animate-spin" /> : <UserX className="h-4 w-4 mr-1" />}
                      Suspenduj
                    </Button>
                  )}
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
import { useState } from "react";
import { useNavigate, useSearchParams } from "react-router-dom";
import { BookOpen, Mail, Lock, User as UserIcon } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Card, CardContent, CardHeader } from "@/components/ui/card";
import { useAuth } from "@/lib/auth-context";
import { toast } from "sonner";

export default function Auth() {
  const [isLogin, setIsLogin] = useState(true);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [fullName, setFullName] = useState("");
  const [loading, setLoading] = useState(false);
  const { signIn, signUp } = useAuth();
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();
  const intent = searchParams.get("intent");

  const getRedirect = () => {
    if (intent === "sell") return "/sell";
    if (intent === "buy") return "/shop";
    return "/";
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      if (isLogin) {
        const { error } = await signIn(email, password);
        if (error) {
          toast.error(error.message);
        } else {
          toast.success("Uspešno ste se prijavili!");
          navigate(getRedirect());
        }
      } else {
        if (!fullName.trim()) {
          toast.error("Unesite vaše ime");
          setLoading(false);
          return;
        }
        const { error } = await signUp(email, password, fullName);
        if (error) {
          toast.error(error.message);
        } else {
          toast.success("Registracija uspešna! Proverite email za potvrdu.");
        }
      }
    } catch (err) {
      toast.error("Došlo je do greške. Pokušajte ponovo.");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="flex min-h-screen items-center justify-center bg-background p-4">
      <Card className="w-full max-w-md shadow-card">
        <CardHeader className="text-center pb-2">
          <div className="mx-auto mb-4 flex h-14 w-14 items-center justify-center rounded-2xl gradient-primary shadow-primary">
            <BookOpen className="h-7 w-7 text-primary-foreground" />
          </div>
          <h1 className="font-display text-2xl font-bold text-foreground">
            {isLogin ? "Prijava" : "Registracija"}
          </h1>
          <p className="text-sm text-muted-foreground">
            {isLogin
              ? intent === "sell"
                ? "Prijavite se da biste prodali knjigu"
                : intent === "buy"
                ? "Prijavite se da biste kupili knjigu"
                : "Prijavite se na svoj BookNook nalog"
              : "Napravite novi BookNook nalog"}
          </p>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            {!isLogin && (
              <div className="space-y-2">
                <Label htmlFor="fullName">Ime i prezime</Label>
                <div className="relative">
                  <UserIcon className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
                  <Input
                    id="fullName"
                    placeholder="Vaše ime"
                    value={fullName}
                    onChange={(e) => setFullName(e.target.value)}
                    className="pl-10"
                  />
                </div>
              </div>
            )}
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <div className="relative">
                <Mail className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
                <Input
                  id="email"
                  type="email"
                  placeholder="vas@email.com"
                  value={email}
                  onChange={(e) => setEmail(e.target.value)}
                  className="pl-10"
                  required
                />
              </div>
            </div>
            <div className="space-y-2">
              <Label htmlFor="password">Lozinka</Label>
              <div className="relative">
                <Lock className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
                <Input
                  id="password"
                  type="password"
                  placeholder="••••••••"
                  value={password}
                  onChange={(e) => setPassword(e.target.value)}
                  className="pl-10"
                  required
                  minLength={6}
                />
              </div>
            </div>
            <Button type="submit" className="w-full" disabled={loading}>
              {loading ? "Molimo sačekajte..." : isLogin ? "Prijavi se" : "Registruj se"}
            </Button>
          </form>
          <div className="mt-4 text-center">
            <button
              type="button"
              onClick={() => setIsLogin(!isLogin)}
              className="text-sm text-primary hover:underline"
            >
              {isLogin ? "Nemate nalog? Registrujte se" : "Već imate nalog? Prijavite se"}
            </button>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
import { useState, useEffect } from "react";
import { useParams, useNavigate } from "react-router-dom";
import { supabase } from "@/integrations/supabase/client";
import { Tables } from "@/integrations/supabase/types";
import { useAuth } from "@/lib/auth-context";
import { Navbar } from "@/components/Navbar";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Label } from "@/components/ui/label";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { BookOpen, ArrowLeft, Loader2, Calendar, MessageCircle, Send, ChevronLeft, ChevronRight, Heart } from "lucide-react";
import { toast } from "sonner";
import { motion, AnimatePresence } from "framer-motion";
import { z } from "zod";

import { SimilarBooks } from "@/components/SimilarBooks";
import { SellerReviews } from "@/components/SellerReviews";
import { useSavedBooks } from "@/hooks/useSavedBooks";
import { ReportListingDialog } from "@/components/ReportListingDialog";

const messageSchema = z.object({
  message: z.string().trim().min(1, "Poruka je obavezna").max(2000),
});

const conditionLabels: Record<string, string> = {
  novo: "Novo",
  "kao novo": "Kao novo",
  dobro: "Dobro",
  korisceno: "Korišćeno",
};

const conditionColors: Record<string, string> = {
  novo: "bg-green-100 text-green-800",
  "kao novo": "bg-blue-100 text-blue-800",
  dobro: "bg-yellow-100 text-yellow-800",
  korisceno: "bg-orange-100 text-orange-800",
};

export default function BookDetails() {
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  const { user } = useAuth();
  const { savedIds, toggleSave } = useSavedBooks();
  const [book, setBook] = useState<Tables<"books"> | null>(null);
  const [sellerName, setSellerName] = useState("");
  const [images, setImages] = useState<string[]>([]);
  const [activeImg, setActiveImg] = useState(0);
  const [loading, setLoading] = useState(true);
  const [sending, setSending] = useState(false);
  const [message, setMessage] = useState("");
  const [error, setError] = useState("");

  useEffect(() => {
    if (!id) return;
    fetchBook();
  }, [id]);

  const fetchBook = async () => {
    setLoading(true);
    const { data, error } = await supabase.from("books").select("*").eq("id", id!).single();
    if (error || !data) {
      toast.error("Knjiga nije pronađena");
      navigate("/");
      return;
    }
    setBook(data);

    const { data: profile } = await supabase
      .from("profiles")
      .select("full_name")
      .eq("user_id", data.seller_id)
      .single();
    setSellerName(profile?.full_name || "Nepoznat prodavac");

    const { data: files } = await supabase.storage
      .from("book-images")
      .list(`${data.seller_id}/${data.id}`);
    if (files && files.length > 0) {
      const urls = files
        .filter((f) => !f.name.startsWith("."))
        .map(
          (f) =>
            supabase.storage
              .from("book-images")
              .getPublicUrl(`${data.seller_id}/${data.id}/${f.name}`).data.publicUrl
        );
      setImages(urls);
    } else if (data.image_url) {
      setImages([data.image_url]);
    }
    setLoading(false);
  };

  const handleSendMessage = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!book || !user) {
      toast.error("Morate biti prijavljeni da biste poslali poruku");
      navigate("/auth");
      return;
    }

    const result = messageSchema.safeParse({ message });
    if (!result.success) {
      setError(result.error.issues[0].message);
      return;
    }
    setError("");
    setSending(true);

    try {
      const { data: existing } = await supabase
        .from("conversations")
        .select("id")
        .eq("book_id", book.id)
        .eq("buyer_id", user.id)
        .eq("seller_id", book.seller_id)
        .single();

      let conversationId: string;

      if (existing) {
        conversationId = existing.id;
      } else {
        const { data: newConv, error: convErr } = await supabase
          .from("conversations")
          .insert({
            book_id: book.id,
            buyer_id: user.id,
            seller_id: book.seller_id,
          })
          .select("id")
          .single();
        if (convErr || !newConv) throw convErr;
        conversationId = newConv.id;
      }

      const { error: msgErr } = await supabase.from("messages").insert({
        conversation_id: conversationId,
        sender_id: user.id,
        message_text: result.data.message,
      });
      if (msgErr) throw msgErr;

      await supabase.from("conversations").update({ last_message_at: new Date().toISOString() }).eq("id", conversationId);

      toast.success("Poruka je poslata!");
      setMessage("");
      navigate(`/conversation/${conversationId}`);
    } catch {
      toast.error("Greška pri slanju poruke");
    }
    setSending(false);
  };

  if (loading) {
    return (
      <div className="min-h-screen bg-background">
        <Navbar />
        <div className="flex justify-center py-32">
          <Loader2 className="h-10 w-10 animate-spin text-primary" />
        </div>
      </div>
    );
  }

  if (!book) return null;

  const isOwnListing = user?.id === book.seller_id;

  return (
    <div className="min-h-screen bg-background">
      <Navbar />
      <div className="container mx-auto px-4 py-8">
        <div className="mb-6 flex items-center justify-between">
          <Button variant="ghost" size="sm" onClick={() => navigate(-1)}>
            <ArrowLeft className="mr-1 h-4 w-4" /> Nazad
          </Button>
          <div className="flex items-center gap-2">
            {user && (
              <Button
                variant="ghost"
                size="sm"
                onClick={() => toggleSave(book.id)}
                className="text-muted-foreground"
              >
                <Heart className={`h-4 w-4 mr-1 ${savedIds.has(book.id) ? "fill-destructive text-destructive" : ""}`} />
                {savedIds.has(book.id) ? "Sačuvano" : "Sačuvaj"}
              </Button>
            )}
            {user && !isOwnListing && (
              <ReportListingDialog bookId={book.id} />
            )}
          </div>
        </div>

        <div className="grid gap-8 lg:grid-cols-2">
          {/* Gallery */}
          <div>
            <div className="relative aspect-[4/3] overflow-hidden rounded-xl bg-secondary">
              {images.length > 0 ? (
                <AnimatePresence mode="wait">
                  <motion.img
                    key={activeImg}
                    src={images[activeImg]}
                    alt={book.title}
                    className="h-full w-full object-contain"
                    initial={{ opacity: 0 }}
                    animate={{ opacity: 1 }}
                    exit={{ opacity: 0 }}
                    transition={{ duration: 0.2 }}
                  />
                </AnimatePresence>
              ) : (
                <div className="flex h-full w-full items-center justify-center">
                  <BookOpen className="h-16 w-16 text-muted-foreground/30" />
                </div>
              )}
              {images.length > 1 && (
                <>
                  <Button variant="secondary" size="icon"
                    className="absolute left-2 top-1/2 h-8 w-8 -translate-y-1/2 rounded-full shadow-md"
                    onClick={() => setActiveImg((p) => (p - 1 + images.length) % images.length)}>
                    <ChevronLeft className="h-4 w-4" />
                  </Button>
                  <Button variant="secondary" size="icon"
                    className="absolute right-2 top-1/2 h-8 w-8 -translate-y-1/2 rounded-full shadow-md"
                    onClick={() => setActiveImg((p) => (p + 1) % images.length)}>
                    <ChevronRight className="h-4 w-4" />
                  </Button>
                </>
              )}
              {book.is_sold && (
                <div className="absolute inset-0 flex items-center justify-center bg-foreground/50">
                  <span className="rounded-lg bg-destructive px-4 py-2 font-display text-lg font-bold text-destructive-foreground">
                    PRODATO
                  </span>
                </div>
              )}
            </div>
            {images.length > 1 && (
              <div className="mt-3 flex gap-2 overflow-x-auto pb-1">
                {images.map((url, i) => (
                  <button key={i} onClick={() => setActiveImg(i)}
                    className={`h-16 w-16 flex-shrink-0 overflow-hidden rounded-lg border-2 transition-colors ${
                      i === activeImg ? "border-primary" : "border-transparent"
                    }`}>
                    <img src={url} alt="" className="h-full w-full object-cover" />
                  </button>
                ))}
              </div>
            )}
          </div>

          {/* Info */}
          <div className="space-y-6">
            <div>
              <h1 className="font-display text-2xl font-bold text-foreground md:text-3xl">{book.title}</h1>
              <p className="mt-1 text-lg text-muted-foreground">{book.author}</p>
            </div>

            <div className="flex items-center gap-3">
              <span className="font-display text-3xl font-bold text-primary">{book.price} RSD</span>
              <span className={`rounded-full px-3 py-1 text-xs font-medium ${conditionColors[book.condition] || ""}`}>
                {conditionLabels[book.condition] || book.condition}
              </span>
              {book.is_sold && <Badge variant="destructive">Prodato</Badge>}
            </div>

            <div className="grid grid-cols-2 gap-3 text-sm">
              <div>
                <span className="text-muted-foreground">Fakultet</span>
                <p className="font-medium text-foreground">{book.faculty}</p>
              </div>
              <div>
                <span className="text-muted-foreground">Predmet</span>
                <p className="font-medium text-foreground">{book.subject}</p>
              </div>
              <div>
                <span className="text-muted-foreground">Godina studija</span>
                <p className="font-medium text-foreground">{book.year_of_study}. godina</p>
              </div>
              <div>
                <span className="text-muted-foreground">Prodavac</span>
                <p className="font-medium text-foreground">{sellerName}</p>
              </div>
            </div>

            <div className="flex items-center gap-2 text-xs text-muted-foreground">
              <Calendar className="h-3.5 w-3.5" />
              Objavljeno {new Date(book.created_at).toLocaleDateString("sr-RS")}
            </div>

            {book.description && (
              <div>
                <h3 className="mb-1 font-display text-sm font-semibold text-foreground">Opis</h3>
                <p className="text-sm leading-relaxed text-muted-foreground">{book.description}</p>
              </div>
            )}

            {/* Contact / Message Form */}
            {!book.is_sold && !isOwnListing ? (
              <Card className="border-primary/20">
                <CardHeader className="pb-3">
                  <CardTitle className="flex items-center gap-2 text-lg">
                    <MessageCircle className="h-5 w-5 text-primary" />
                    Pošalji poruku prodavcu
                  </CardTitle>
                </CardHeader>
                <CardContent>
                  <form onSubmit={handleSendMessage} className="space-y-3">
                    <div>
                      <Label htmlFor="message">Poruka</Label>
                      <Textarea
                        id="message"
                        value={message}
                        onChange={(e) => setMessage(e.target.value)}
                        placeholder="Napišite poruku prodavcu..."
                        rows={3}
                        maxLength={2000}
                      />
                      {error && <p className="mt-1 text-xs text-destructive">{error}</p>}
                    </div>
                    <Button type="submit" className="w-full" disabled={sending}>
                      {sending ? <Loader2 className="mr-2 h-4 w-4 animate-spin" /> : <Send className="mr-2 h-4 w-4" />}
                      Pošalji poruku
                    </Button>
                    {!user && (
                      <p className="text-xs text-muted-foreground text-center">
                        Morate biti prijavljeni da biste poslali poruku.
                      </p>
                    )}
                  </form>
                </CardContent>
              </Card>
            ) : book.is_sold ? (
              <Card className="border-destructive/20 bg-destructive/5">
                <CardContent className="py-6 text-center">
                  <p className="font-display font-semibold text-destructive">Ovaj oglas je zatvoren — knjiga je prodata.</p>
                </CardContent>
              </Card>
            ) : null}
          </div>
        </div>

        {/* Seller Reviews */}
        <div className="mt-8">
          <SellerReviews sellerId={book.seller_id} />
        </div>

        {/* Similar Books */}
        <SimilarBooks book={book} />
      </div>
    </div>
  );
}
import { useState, useRef, useEffect } from "react";
import { Send, Bot, User, BookOpen } from "lucide-react";
import chatBg from "@/assets/chat-bg.jpg";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Navbar } from "@/components/Navbar";
import ReactMarkdown from "react-markdown";
import { supabase } from "@/integrations/supabase/client";

type Msg = { role: "user" | "assistant"; content: string };

const CHAT_URL = `${import.meta.env.VITE_SUPABASE_URL}/functions/v1/book-chat`;

export default function Chatbot() {
  const [messages, setMessages] = useState<Msg[]>([]);
  const [input, setInput] = useState("");
  const [isLoading, setIsLoading] = useState(false);
  const bottomRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  const send = async () => {
    if (!input.trim() || isLoading) return;
    const userMsg: Msg = { role: "user", content: input.trim() };
    setInput("");
    setMessages((prev) => [...prev, userMsg]);
    setIsLoading(true);

    let assistantSoFar = "";
    const allMessages = [...messages, userMsg];

    try {
      const resp = await fetch(CHAT_URL, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY}`,
        },
        body: JSON.stringify({ messages: allMessages }),
      });

      if (!resp.ok || !resp.body) {
        if (resp.status === 429) {
          setMessages((prev) => [...prev, { role: "assistant", content: "Previše zahteva. Pokušajte ponovo za minut." }]);
        } else {
          setMessages((prev) => [...prev, { role: "assistant", content: "Greška. Pokušajte ponovo." }]);
        }
        setIsLoading(false);
        return;
      }

      const reader = resp.body.getReader();
      const decoder = new TextDecoder();
      let textBuffer = "";

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        textBuffer += decoder.decode(value, { stream: true });

        let newlineIndex: number;
        while ((newlineIndex = textBuffer.indexOf("\n")) !== -1) {
          let line = textBuffer.slice(0, newlineIndex);
          textBuffer = textBuffer.slice(newlineIndex + 1);
          if (line.endsWith("\r")) line = line.slice(0, -1);
          if (line.startsWith(":") || line.trim() === "") continue;
          if (!line.startsWith("data: ")) continue;
          const jsonStr = line.slice(6).trim();
          if (jsonStr === "[DONE]") break;
          try {
            const parsed = JSON.parse(jsonStr);
            const content = parsed.choices?.[0]?.delta?.content;
            if (content) {
              assistantSoFar += content;
              setMessages((prev) => {
                const last = prev[prev.length - 1];
                if (last?.role === "assistant") {
                  return prev.map((m, i) => i === prev.length - 1 ? { ...m, content: assistantSoFar } : m);
                }
                return [...prev, { role: "assistant", content: assistantSoFar }];
              });
            }
          } catch {}
        }
      }
    } catch {
      setMessages((prev) => [...prev, { role: "assistant", content: "Greška u komunikaciji. Pokušajte ponovo." }]);
    }
    setIsLoading(false);
  };

  return (
    <div className="relative flex min-h-screen flex-col">
      <div className="fixed inset-0 -z-10 bg-cover bg-center bg-no-repeat" style={{ backgroundImage: `url(${chatBg})` }} />
      <div className="fixed inset-0 -z-10 bg-background/80 backdrop-blur-sm" />
      <Navbar />
      <div className="flex flex-1 flex-col container mx-auto max-w-3xl px-4 py-4">
        <div className="mb-4 text-center">
          <div className="mx-auto mb-2 flex h-12 w-12 items-center justify-center rounded-2xl gradient-primary">
            <Bot className="h-6 w-6 text-primary-foreground" />
          </div>
          <h1 className="font-display text-xl font-bold text-foreground">AI Pomoćnik</h1>
          <p className="text-sm text-muted-foreground">Pitajte me za preporuku literature za vaš fakultet</p>
        </div>

        <div className="flex-1 space-y-4 overflow-y-auto rounded-xl border border-border bg-card p-4 mb-4" style={{ maxHeight: "60vh" }}>
          {messages.length === 0 && (
            <div className="flex flex-col items-center justify-center py-12 text-center">
              <BookOpen className="h-10 w-10 text-muted-foreground/30 mb-3" />
              <p className="text-muted-foreground text-sm">Započnite razgovor pitanjem o knjigama...</p>
              <div className="mt-4 flex flex-wrap gap-2 justify-center">
                {["Koja literatura za Pravni?", "Preporuči udžbenike za programiranje", "Šta je obavezna literatura za 2. godinu ETF?"].map((q) => (
                  <button key={q} onClick={() => { setInput(q); }} className="rounded-full border border-border bg-secondary px-3 py-1.5 text-xs text-secondary-foreground hover:bg-accent transition-colors">
                    {q}
                  </button>
                ))}
              </div>
            </div>
          )}
          {messages.map((msg, i) => (
            <div key={i} className={`flex gap-3 ${msg.role === "user" ? "justify-end" : "justify-start"}`}>
              {msg.role === "assistant" && (
                <div className="flex h-8 w-8 shrink-0 items-center justify-center rounded-lg gradient-primary">
                  <Bot className="h-4 w-4 text-primary-foreground" />
                </div>
              )}
              <div className={`max-w-[80%] rounded-2xl px-4 py-3 text-sm ${msg.role === "user" ? "bg-primary text-primary-foreground" : "bg-secondary text-secondary-foreground"}`}>
                {msg.role === "assistant" ? (
                  <div className="prose prose-sm max-w-none">
                    <ReactMarkdown>{msg.content}</ReactMarkdown>
                  </div>
                ) : msg.content}
              </div>
              {msg.role === "user" && (
                <div className="flex h-8 w-8 shrink-0 items-center justify-center rounded-lg bg-muted">
                  <User className="h-4 w-4 text-muted-foreground" />
                </div>
              )}
            </div>
          ))}
          {isLoading && messages[messages.length - 1]?.role !== "assistant" && (
            <div className="flex gap-3">
              <div className="flex h-8 w-8 shrink-0 items-center justify-center rounded-lg gradient-primary">
                <Bot className="h-4 w-4 text-primary-foreground" />
              </div>
              <div className="rounded-2xl bg-secondary px-4 py-3">
                <div className="flex gap-1">
                  <span className="h-2 w-2 animate-bounce rounded-full bg-muted-foreground/40" style={{ animationDelay: "0ms" }} />
                  <span className="h-2 w-2 animate-bounce rounded-full bg-muted-foreground/40" style={{ animationDelay: "150ms" }} />
                  <span className="h-2 w-2 animate-bounce rounded-full bg-muted-foreground/40" style={{ animationDelay: "300ms" }} />
                </div>
              </div>
            </div>
          )}
          <div ref={bottomRef} />
        </div>

        <div className="flex gap-2">
          <Input
            value={input}
            onChange={(e) => setInput(e.target.value)}
            onKeyDown={(e) => e.key === "Enter" && send()}
            placeholder="Pitajte o literaturi..."
            className="bg-card"
            disabled={isLoading}
          />
          <Button onClick={send} disabled={isLoading || !input.trim()} size="icon">
            <Send className="h-4 w-4" />
          </Button>
        </div>
      </div>
    </div>
  );
}
import { useState } from "react";
import { useNavigate } from "react-router-dom";
import { Navbar } from "@/components/Navbar";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";
import { supabase } from "@/integrations/supabase/client";
import { useAuth } from "@/lib/auth-context";
import { toast } from "@/components/ui/sonner";
import { Mail, Phone, Instagram, Send, CheckCircle } from "lucide-react";
import { z } from "zod";

const contactSchema = z.object({
  full_name: z.string().trim().min(1, "Ime je obavezno").max(100),
  email: z.string().trim().email("Nevažeća email adresa").max(255),
  subject: z.string().trim().min(1, "Predmet je obavezan").max(200),
  message: z.string().trim().min(1, "Poruka je obavezna").max(2000),
});

export default function Contact() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const [form, setForm] = useState({ full_name: "", email: "", subject: "", message: "" });
  const [loading, setLoading] = useState(false);
  const [sent, setSent] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const parsed = contactSchema.safeParse(form);
    if (!parsed.success) {
      toast.error(parsed.error.errors[0].message);
      return;
    }
    setLoading(true);
    const { full_name, email, subject, message } = parsed.data;

    // Always save to contact_messages for backwards compatibility
    await supabase.from("contact_messages").insert([{ full_name, email, subject, message }]);

    // If user is logged in, also create a support conversation thread
    if (user) {
      const { data: convo } = await supabase
        .from("support_conversations")
        .insert([{ user_id: user.id, subject } as any])
        .select()
        .single();

      if (convo) {
        await supabase.from("support_messages").insert([{
          conversation_id: (convo as any).id,
          sender_id: user.id,
          message_text: message,
          is_admin: false,
        } as any]);
      }
    }

    setLoading(false);
    setSent(true);
  };

  return (
    <div className="min-h-screen bg-background">
      <Navbar />
      <div className="container mx-auto px-4 py-10">
        <div className="mx-auto max-w-5xl">
          <h1 className="font-display text-3xl font-bold text-foreground mb-2">Kontakt</h1>
          <p className="text-muted-foreground mb-8">Pošaljite nam poruku ili nas kontaktirajte putem drugih kanala.</p>

          <div className="grid gap-8 md:grid-cols-5">
            {/* Form */}
            <div className="md:col-span-3">
              <Card className="border-border">
                <CardHeader>
                  <CardTitle className="text-lg">Pošaljite poruku</CardTitle>
                </CardHeader>
                <CardContent>
                  {sent ? (
                    <div className="flex flex-col items-center gap-4 py-10 text-center">
                      <div className="flex h-16 w-16 items-center justify-center rounded-full bg-primary/10">
                        <CheckCircle className="h-8 w-8 text-primary" />
                      </div>
                      <p className="text-lg font-semibold text-foreground">Vaša poruka je uspešno poslata!</p>
                      <p className="text-muted-foreground">BookNook tim će vas kontaktirati uskoro.</p>
                      <div className="flex gap-3">
                        <Button variant="outline" onClick={() => { setSent(false); setForm({ full_name: "", email: "", subject: "", message: "" }); }}>
                          Pošalji novu poruku
                        </Button>
                        {user && (
                          <Button onClick={() => navigate("/support")}>
                            Moji upiti
                          </Button>
                        )}
                      </div>
                    </div>
                  ) : (
                    <form onSubmit={handleSubmit} className="space-y-4">
                      <div className="space-y-2">
                        <Label htmlFor="full_name">Ime i prezime</Label>
                        <Input id="full_name" value={form.full_name} onChange={e => setForm(f => ({ ...f, full_name: e.target.value }))} placeholder="Vaše ime" required />
                      </div>
                      <div className="space-y-2">
                        <Label htmlFor="email">Email</Label>
                        <Input id="email" type="email" value={form.email} onChange={e => setForm(f => ({ ...f, email: e.target.value }))} placeholder="vaš@email.com" required />
                      </div>
                      <div className="space-y-2">
                        <Label htmlFor="subject">Predmet</Label>
                        <Input id="subject" value={form.subject} onChange={e => setForm(f => ({ ...f, subject: e.target.value }))} placeholder="O čemu se radi?" required />
                      </div>
                      <div className="space-y-2">
                        <Label htmlFor="message">Poruka</Label>
                        <Textarea id="message" value={form.message} onChange={e => setForm(f => ({ ...f, message: e.target.value }))} placeholder="Opišite vaš upit..." rows={5} required />
                      </div>
                      <Button type="submit" disabled={loading} className="w-full gradient-primary border-0">
                        <Send className="mr-2 h-4 w-4" />
                        {loading ? "Slanje..." : "Pošalji poruku"}
                      </Button>
                    </form>
                  )}
                </CardContent>
              </Card>
            </div>

            {/* Contact info */}
            <div className="md:col-span-2 space-y-4">
              <Card className="border-border">
                <CardHeader>
                  <CardTitle className="text-lg">Ostali kontakti</CardTitle>
                </CardHeader>
                <CardContent className="space-y-5">
                  <a href="mailto:booknooksupport@gmail.com" className="flex items-center gap-3 text-foreground hover:text-primary transition-colors">
                    <div className="flex h-10 w-10 shrink-0 items-center justify-center rounded-lg bg-primary/10">
                      <Mail className="h-5 w-5 text-primary" />
                    </div>
                    <div>
                      <p className="text-xs text-muted-foreground">Email</p>
                      <p className="text-sm font-medium">booknooksupport@gmail.com</p>
                    </div>
                  </a>
                  <a href="tel:+381641234567" className="flex items-center gap-3 text-foreground hover:text-primary transition-colors">
                    <div className="flex h-10 w-10 shrink-0 items-center justify-center rounded-lg bg-primary/10">
                      <Phone className="h-5 w-5 text-primary" />
                    </div>
                    <div>
                      <p className="text-xs text-muted-foreground">Telefon</p>
                      <p className="text-sm font-medium">+381 64 123 4567</p>
                    </div>
                  </a>
                  <a href="https://instagram.com/booknook.rs" target="_blank" rel="noopener noreferrer" className="flex items-center gap-3 text-foreground hover:text-primary transition-colors">
                    <div className="flex h-10 w-10 shrink-0 items-center justify-center rounded-lg bg-primary/10">
                      <Instagram className="h-5 w-5 text-primary" />
                    </div>
                    <div>
                      <p className="text-xs text-muted-foreground">Instagram</p>
                      <p className="text-sm font-medium">booknook.rs</p>
                    </div>
                  </a>
                </CardContent>
              </Card>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}
import { useState, useEffect, useRef } from "react";
import { useParams, useNavigate, Link } from "react-router-dom";
import { Loader2, ArrowLeft, Send, BookOpen, Star } from "lucide-react";
import chatBg from "@/assets/chat-bg.jpg";
import { Navbar } from "@/components/Navbar";
import { useAuth } from "@/lib/auth-context";
import { supabase } from "@/integrations/supabase/client";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { ReviewDialog } from "@/components/ReviewDialog";
import { format, formatDistanceToNow } from "date-fns";
import { sr } from "date-fns/locale";

interface Message {
  id: string;
  sender_id: string;
  message_text: string;
  created_at: string;
  is_read: boolean;
}

interface ConversationInfo {
  id: string;
  book_id: string;
  buyer_id: string;
  seller_id: string;
  book_title: string;
  other_name: string;
  other_last_seen: string | null;
}

export default function Conversation() {
  const { id } = useParams<{ id: string }>();
  const { user } = useAuth();
  const navigate = useNavigate();
  const [info, setInfo] = useState<ConversationInfo | null>(null);
  const [messages, setMessages] = useState<Message[]>([]);
  const [loading, setLoading] = useState(true);
  const [sending, setSending] = useState(false);
  const [text, setText] = useState("");
  const bottomRef = useRef<HTMLDivElement>(null);
  const [hasReview, setHasReview] = useState(true); // default true to hide button until checked
  const [canReview, setCanReview] = useState(false);

  useEffect(() => {
    if (!user) { navigate("/auth"); return; }
    if (!id) return;
    fetchConversation();
  }, [user, id]);

  useEffect(() => {
    if (!id) return;
    const channel = supabase
      .channel(`conv-${id}`)
      .on("postgres_changes", {
        event: "INSERT",
        schema: "public",
        table: "messages",
        filter: `conversation_id=eq.${id}`,
      }, (payload) => {
        const newMsg = payload.new as Message;
        setMessages((prev) => [...prev, newMsg]);
        if (user && newMsg.sender_id !== user.id) {
          supabase.from("messages").update({ is_read: true }).eq("id", newMsg.id).then();
        }
      })
      .subscribe();
    return () => { supabase.removeChannel(channel); };
  }, [id, user]);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  const fetchConversation = async () => {
    setLoading(true);
    const { data: conv } = await supabase
      .from("conversations")
      .select("id, book_id, buyer_id, seller_id, books(title)")
      .eq("id", id!)
      .single();

    if (!conv) { navigate("/inbox"); return; }

    const otherId = conv.buyer_id === user!.id ? conv.seller_id : conv.buyer_id;
    const { data: profile } = await supabase
      .from("profiles")
      .select("full_name, last_seen_at")
      .eq("user_id", otherId)
      .single();

    setInfo({
      id: conv.id,
      book_id: conv.book_id,
      buyer_id: conv.buyer_id,
      seller_id: conv.seller_id,
      book_title: (conv.books as any)?.title || "Knjiga",
      other_name: profile?.full_name || "Korisnik",
      other_last_seen: profile?.last_seen_at || null,
    });

    const { data: msgs } = await supabase
      .from("messages")
      .select("*")
      .eq("conversation_id", id!)
      .order("created_at", { ascending: true });

    setMessages(msgs || []);

    if (msgs && user) {
      const unreadIds = msgs.filter((m) => !m.is_read && m.sender_id !== user.id).map((m) => m.id);
      if (unreadIds.length > 0) {
        await supabase.from("messages").update({ is_read: true }).in("id", unreadIds);
      }
    }

    // Check if user already reviewed and if eligible
    if (user) {
      const sellerId = conv.buyer_id === user.id ? conv.seller_id : conv.buyer_id;
      // Only buyers can review sellers
      if (conv.buyer_id === user.id) {
        const { data: existingReview } = await supabase
          .from("reviews")
          .select("id")
          .eq("reviewer_id", user.id)
          .eq("conversation_id", id!)
          .maybeSingle();
        setHasReview(!!existingReview);
        
        // Check that at least one message was sent by this user
        const hasSentMsg = msgs?.some((m) => m.sender_id === user.id);
        setCanReview(!!hasSentMsg && !existingReview);
      } else {
        setHasReview(true);
        setCanReview(false);
      }
    }

    setLoading(false);
  };

  const handleSend = async () => {
    if (!text.trim() || !user || !id || sending) return;
    setSending(true);
    const { error } = await supabase.from("messages").insert({
      conversation_id: id,
      sender_id: user.id,
      message_text: text.trim(),
    });
    if (!error) {
      setText("");
      await supabase.from("conversations").update({ last_message_at: new Date().toISOString() }).eq("id", id);
    }
    setSending(false);
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault();
      handleSend();
    }
  };

  const getOnlineStatus = () => {
    if (!info?.other_last_seen) return null;
    const lastSeen = new Date(info.other_last_seen);
    const diffMs = Date.now() - lastSeen.getTime();
    const isOnline = diffMs < 5 * 60 * 1000; // 5 minutes
    if (isOnline) {
      return <span className="flex items-center gap-1 text-xs text-green-600"><span className="h-2 w-2 rounded-full bg-green-500" />Na mreži</span>;
    }
    return <span className="text-xs text-muted-foreground">Poslednji put viđen/a {formatDistanceToNow(lastSeen, { addSuffix: true, locale: sr })}</span>;
  };

  if (!user) return null;

  return (
    <div className="relative flex min-h-screen flex-col">
      <div className="fixed inset-0 -z-10 bg-cover bg-center bg-no-repeat" style={{ backgroundImage: `url(${chatBg})` }} />
      <div className="fixed inset-0 -z-10 bg-background/80 backdrop-blur-sm" />
      <Navbar />
      {loading ? (
        <div className="flex flex-1 items-center justify-center">
          <Loader2 className="h-8 w-8 animate-spin text-primary" />
        </div>
      ) : info ? (
        <div className="flex flex-1 flex-col">
          {/* Header */}
          <div className="sticky top-16 z-40 border-b border-border bg-card/90 backdrop-blur-sm">
            <div className="container mx-auto flex items-center gap-3 px-4 py-3 max-w-2xl">
              <Button variant="ghost" size="icon" onClick={() => navigate("/inbox")}>
                <ArrowLeft className="h-4 w-4" />
              </Button>
              <div className="min-w-0 flex-1">
                <div className="flex items-center gap-2">
                  <p className="truncate font-medium text-foreground">{info.other_name}</p>
                  {getOnlineStatus()}
                </div>
                <Link to={`/listing/${info.book_id}`} className="flex items-center gap-1 text-xs text-primary hover:underline">
                  <BookOpen className="h-3 w-3" />
                  {info.book_title}
                </Link>
              </div>
              {canReview && info && (
                <ReviewDialog
                  sellerId={info.seller_id}
                  reviewerId={user.id}
                  bookId={info.book_id}
                  conversationId={info.id}
                  onReviewSubmitted={() => { setCanReview(false); setHasReview(true); }}
                />
              )}
              {hasReview && info && user.id === info.buyer_id && (
                <span className="flex items-center gap-1 text-xs text-muted-foreground">
                  <Star className="h-3 w-3 fill-yellow-400 text-yellow-400" /> Ocenjeno
                </span>
              )}
            </div>
          </div>

          {/* Messages */}
          <div className="flex-1 overflow-y-auto">
            <div className="container mx-auto max-w-2xl px-4 py-4 space-y-3">
              {messages.length === 0 && (
                <p className="text-center text-sm text-muted-foreground py-10">Započnite razgovor.</p>
              )}
              {messages.map((msg) => {
                const isMe = msg.sender_id === user.id;
                return (
                  <div key={msg.id} className={`flex ${isMe ? "justify-end" : "justify-start"}`}>
                    <div className={`max-w-[75%] rounded-2xl px-4 py-2.5 text-sm ${
                      isMe
                        ? "bg-primary text-primary-foreground rounded-br-md"
                        : "bg-secondary text-secondary-foreground rounded-bl-md"
                    }`}>
                      <p className="whitespace-pre-wrap break-words">{msg.message_text}</p>
                      <p className={`mt-1 text-[10px] ${isMe ? "text-primary-foreground/60" : "text-muted-foreground"}`}>
                        {format(new Date(msg.created_at), "HH:mm")}
                      </p>
                    </div>
                  </div>
                );
              })}
              <div ref={bottomRef} />
            </div>
          </div>

          {/* Input */}
          <div className="sticky bottom-0 border-t border-border bg-card">
            <div className="container mx-auto flex items-end gap-2 px-4 py-3 max-w-2xl">
              <Textarea
                value={text}
                onChange={(e) => setText(e.target.value)}
                onKeyDown={handleKeyDown}
                placeholder="Napišite poruku..."
                rows={1}
                className="min-h-[40px] max-h-32 resize-none"
              />
              <Button size="icon" onClick={handleSend} disabled={!text.trim() || sending}>
                {sending ? <Loader2 className="h-4 w-4 animate-spin" /> : <Send className="h-4 w-4" />}
              </Button>
            </div>
          </div>
        </div>
      ) : null}
    </div>
  );
}
import { useState, useEffect } from "react";
import { useNavigate, useParams } from "react-router-dom";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Card, CardContent, CardHeader } from "@/components/ui/card";
import { Navbar } from "@/components/Navbar";
import { ImageUpload } from "@/components/ImageUpload";
import { FacultySelect } from "@/components/FacultySelect";
import { useAuth } from "@/lib/auth-context";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";
import { Loader2 } from "lucide-react";
import { Switch } from "@/components/ui/switch";


export default function EditBook() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const { id: bookId } = useParams<{ id: string }>();

  const [loading, setLoading] = useState(true);
  const [saving, setSaving] = useState(false);

  // New files to upload
  const [newImages, setNewImages] = useState<File[]>([]);
  const [newPreviews, setNewPreviews] = useState<string[]>([]);

  // Existing remote images (URLs)
  const [existingUrls, setExistingUrls] = useState<string[]>([]);

  const [form, setForm] = useState({
    title: "", author: "", faculty: "", subject: "",
    year_of_study: "", price: "", condition: "", description: "", is_sold: false,
  });

  const update = (key: string, value: string | boolean) => setForm((p) => ({ ...p, [key]: value }));

  // Load book data + existing images
  useEffect(() => {
    if (!user) { navigate("/auth"); return; }
    if (!bookId) { navigate("/profile"); return; }

    const load = async () => {
      const { data: book, error } = await supabase
        .from("books").select("*").eq("id", bookId).single();

      if (error || !book) { toast.error("Knjiga nije pronađena"); navigate("/profile"); return; }
      if (book.seller_id !== user.id) { toast.error("Nemate dozvolu"); navigate("/profile"); return; }

      setForm({
        title: book.title,
        author: book.author,
        faculty: book.faculty,
        subject: book.subject,
        year_of_study: String(book.year_of_study),
        price: String(book.price),
        condition: book.condition,
        description: book.description || "",
        is_sold: book.is_sold,
      });

      // List existing images in storage
      const { data: files } = await supabase.storage
        .from("book-images").list(`${user.id}/${bookId}`);
      if (files && files.length > 0) {
        const urls = files
          .sort((a, b) => a.name.localeCompare(b.name))
          .map((f) => {
            const { data } = supabase.storage.from("book-images").getPublicUrl(`${user.id}/${bookId}/${f.name}`);
            return data.publicUrl;
          });
        setExistingUrls(urls);
      }

      setLoading(false);
    };
    load();
  }, [user, bookId]);

  const handleAddImages = (files: File[]) => {
    const totalCount = existingUrls.length + newImages.length + files.length;
    const allowed = Math.max(0, 5 - existingUrls.length - newImages.length);
    const toAdd = files.slice(0, allowed);
    setNewImages((p) => [...p, ...toAdd]);
    setNewPreviews((p) => [...p, ...toAdd.map((f) => URL.createObjectURL(f))]);
  };

  const handleRemoveExisting = async (index: number) => {
    const url = existingUrls[index];
    // Extract path from URL
    const bucketBase = supabase.storage.from("book-images").getPublicUrl("").data.publicUrl;
    const path = url.replace(bucketBase, "");
    await supabase.storage.from("book-images").remove([path]);
    setExistingUrls((p) => p.filter((_, i) => i !== index));
  };

  const handleRemoveNew = (index: number) => {
    URL.revokeObjectURL(newPreviews[index]);
    setNewImages((p) => p.filter((_, i) => i !== index));
    setNewPreviews((p) => p.filter((_, i) => i !== index));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!user || !bookId) return;
    setSaving(true);

    try {
      // Upload new images
      const startIndex = existingUrls.length;
      const uploadedUrls: string[] = [];

      for (let i = 0; i < newImages.length; i++) {
        const file = newImages[i];
        const ext = file.name.split(".").pop();
        const path = `${user.id}/${bookId}/${startIndex + i}.${ext}`;
        const { error } = await supabase.storage.from("book-images").upload(path, file, { upsert: true });
        if (error) throw error;
        const { data } = supabase.storage.from("book-images").getPublicUrl(path);
        uploadedUrls.push(data.publicUrl);
      }

      // Determine primary image
      const allUrls = [...existingUrls, ...uploadedUrls];
      const imageUrl = allUrls.length > 0 ? allUrls[0] : null;

      const { error } = await supabase.from("books").update({
        title: form.title,
        author: form.author,
        faculty: form.faculty,
        subject: form.subject,
        year_of_study: parseInt(form.year_of_study),
        price: parseFloat(form.price),
        condition: form.condition,
        description: form.description || null,
        is_sold: form.is_sold,
        image_url: imageUrl,
      }).eq("id", bookId);

      if (error) throw error;

      toast.success("Knjiga je uspešno ažurirana!");
      navigate("/profile");
    } catch (err) {
      toast.error("Greška pri ažuriranju knjige");
      console.error(err);
    } finally {
      setSaving(false);
    }
  };

  if (!user) return null;

  if (loading) {
    return (
      <div className="min-h-screen bg-background">
        <Navbar />
        <div className="flex justify-center py-20">
          <Loader2 className="h-8 w-8 animate-spin text-primary" />
        </div>
      </div>
    );
  }

  const totalImages = existingUrls.length + newImages.length;
  const allPreviews = [...existingUrls, ...newPreviews];

  return (
    <div className="min-h-screen bg-background">
      <Navbar />
      <div className="container mx-auto max-w-2xl px-4 py-8">
        <Card className="shadow-card">
          <CardHeader>
            <h1 className="font-display text-2xl font-bold text-foreground">Izmeni oglas</h1>
            <p className="text-sm text-muted-foreground">Ažurirajte informacije o knjizi</p>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleSubmit} className="space-y-4">
              {/* Images */}
              <div className="space-y-2">
                <Label>Fotografije knjige</Label>
                <div className="space-y-2">
                  <div className="grid grid-cols-3 gap-3 sm:grid-cols-5">
                    {/* Existing images */}
                    {existingUrls.map((src, i) => (
                      <div key={`existing-${i}`} className="group relative aspect-square overflow-hidden rounded-lg border border-border bg-muted">
                        <img src={src} alt={`Slika ${i + 1}`} className="h-full w-full object-cover" />
                        {!saving && (
                          <button
                            type="button"
                            onClick={() => handleRemoveExisting(i)}
                            className="absolute right-1 top-1 rounded-full bg-destructive p-1 text-destructive-foreground opacity-0 transition-opacity group-hover:opacity-100"
                          >
                            <span className="block h-3 w-3 text-xs leading-3">✕</span>
                          </button>
                        )}
                      </div>
                    ))}
                    {/* New image previews */}
                    {newPreviews.map((src, i) => (
                      <div key={`new-${i}`} className="group relative aspect-square overflow-hidden rounded-lg border border-border bg-muted">
                        <img src={src} alt={`Nova slika ${i + 1}`} className="h-full w-full object-cover" />
                        {!saving && (
                          <button
                            type="button"
                            onClick={() => handleRemoveNew(i)}
                            className="absolute right-1 top-1 rounded-full bg-destructive p-1 text-destructive-foreground opacity-0 transition-opacity group-hover:opacity-100"
                          >
                            <span className="block h-3 w-3 text-xs leading-3">✕</span>
                          </button>
                        )}
                      </div>
                    ))}
                    {/* Add button */}
                    {totalImages < 5 && !saving && (
                      <label className="flex aspect-square cursor-pointer flex-col items-center justify-center gap-1 rounded-lg border-2 border-dashed border-muted-foreground/30 bg-muted/50 text-muted-foreground transition-colors hover:border-primary hover:text-primary">
                        <span className="text-xl">+</span>
                        <span className="text-[10px] font-medium">{totalImages}/5</span>
                        <input
                          type="file"
                          accept="image/*"
                          multiple
                          className="hidden"
                          onChange={(e) => {
                            const files = Array.from(e.target.files || []);
                            handleAddImages(files);
                            e.target.value = "";
                          }}
                        />
                      </label>
                    )}
                  </div>
                  <p className="text-xs text-muted-foreground">Dodajte do 5 fotografija knjige (JPG, PNG)</p>
                </div>
              </div>

              <div className="grid gap-4 sm:grid-cols-2">
                <div className="space-y-2">
                  <Label>Naslov knjige *</Label>
                  <Input value={form.title} onChange={(e) => update("title", e.target.value)} required />
                </div>
                <div className="space-y-2">
                  <Label>Autor *</Label>
                  <Input value={form.author} onChange={(e) => update("author", e.target.value)} required />
                </div>
              </div>
              <div className="grid gap-4 sm:grid-cols-2">
                <div className="space-y-2">
                  <Label>Fakultet *</Label>
                  <FacultySelect value={form.faculty} onValueChange={(v) => update("faculty", v)} disabled={saving} />
                </div>
                <div className="space-y-2">
                  <Label>Predmet *</Label>
                  <Input value={form.subject} onChange={(e) => update("subject", e.target.value)} required />
                </div>
              </div>
              <div className="grid gap-4 sm:grid-cols-3">
                <div className="space-y-2">
                  <Label>Godina studija *</Label>
                  <Select value={form.year_of_study} onValueChange={(v) => update("year_of_study", v)}>
                    <SelectTrigger><SelectValue placeholder="Godina" /></SelectTrigger>
                    <SelectContent>
                      {[1,2,3,4,5,6].map((y) => <SelectItem key={y} value={String(y)}>{y}. godina</SelectItem>)}
                    </SelectContent>
                  </Select>
                </div>
                <div className="space-y-2">
                  <Label>Cena (RSD) *</Label>
                  <Input type="number" min="0" value={form.price} onChange={(e) => update("price", e.target.value)} required />
                </div>
                <div className="space-y-2">
                  <Label>Stanje *</Label>
                  <Select value={form.condition} onValueChange={(v) => update("condition", v)}>
                    <SelectTrigger><SelectValue placeholder="Stanje" /></SelectTrigger>
                    <SelectContent>
                      <SelectItem value="novo">Novo</SelectItem>
                      <SelectItem value="kao novo">Kao novo</SelectItem>
                      <SelectItem value="dobro">Dobro</SelectItem>
                      <SelectItem value="korisceno">Korišćeno</SelectItem>
                    </SelectContent>
                  </Select>
                </div>
              </div>
              <div className="space-y-2">
                <Label>Opis (opciono)</Label>
                <Textarea value={form.description} onChange={(e) => update("description", e.target.value)} rows={3} />
              </div>

              {/* Sold status */}
              <div className="flex items-center gap-3 rounded-lg border border-border p-3">
                <Switch checked={form.is_sold} onCheckedChange={(v) => update("is_sold", v)} id="is_sold" />
                <Label htmlFor="is_sold" className="cursor-pointer">
                  {form.is_sold ? "Prodato" : "Dostupno"}
                </Label>
              </div>

              <div className="flex gap-3">
                <Button type="button" variant="outline" className="flex-1" onClick={() => navigate("/profile")} disabled={saving}>
                  Otkaži
                </Button>
                <Button type="submit" className="flex-1" disabled={saving}>
                  {saving ? "Čuvanje..." : "Sačuvaj izmene"}
                </Button>
              </div>
            </form>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
import { useState, useEffect } from "react";
import { useNavigate } from "react-router-dom";
import { Loader2, MessageCircle } from "lucide-react";
import chatBg from "@/assets/chat-bg.jpg";
import { Navbar } from "@/components/Navbar";
import { useAuth } from "@/lib/auth-context";
import { supabase } from "@/integrations/supabase/client";
import { Card, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { formatDistanceToNow } from "date-fns";
import { sr } from "date-fns/locale";

interface ConversationPreview {
  id: string;
  book_id: string;
  buyer_id: string;
  seller_id: string;
  last_message_at: string;
  book_title: string;
  other_name: string;
  last_message: string;
  unread_count: number;
}

export default function Inbox() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const [conversations, setConversations] = useState<ConversationPreview[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!user) {
      navigate("/auth");
      return;
    }
    fetchConversations();

    // Realtime subscription for new messages
    const channel = supabase
      .channel("inbox-updates")
      .on("postgres_changes", { event: "*", schema: "public", table: "messages" }, () => {
        fetchConversations();
      })
      .on("postgres_changes", { event: "UPDATE", schema: "public", table: "conversations" }, () => {
        fetchConversations();
      })
      .subscribe();

    return () => { supabase.removeChannel(channel); };
  }, [user]);

  const fetchConversations = async () => {
    if (!user) return;
    setLoading(true);

    const { data: convos } = await supabase
      .from("conversations")
      .select("id, book_id, buyer_id, seller_id, last_message_at, books(title)")
      .order("last_message_at", { ascending: false });

    if (!convos || convos.length === 0) {
      setConversations([]);
      setLoading(false);
      return;
    }

    // Get other participant names and last messages
    const otherIds = convos.map((c: any) => c.buyer_id === user.id ? c.seller_id : c.buyer_id);
    const uniqueIds = [...new Set(otherIds)];

    const { data: profiles } = await supabase
      .from("profiles")
      .select("user_id, full_name")
      .in("user_id", uniqueIds);

    const profileMap: Record<string, string> = {};
    profiles?.forEach((p: any) => { profileMap[p.user_id] = p.full_name || "Korisnik"; });

    // Fetch last message and unread count per conversation
    const previews: ConversationPreview[] = [];

    for (const c of convos as any[]) {
      const { data: lastMsg } = await supabase
        .from("messages")
        .select("message_text")
        .eq("conversation_id", c.id)
        .order("created_at", { ascending: false })
        .limit(1)
        .single();

      const { count } = await supabase
        .from("messages")
        .select("id", { count: "exact", head: true })
        .eq("conversation_id", c.id)
        .eq("is_read", false)
        .neq("sender_id", user.id);

      const otherId = c.buyer_id === user.id ? c.seller_id : c.buyer_id;

      previews.push({
        id: c.id,
        book_id: c.book_id,
        buyer_id: c.buyer_id,
        seller_id: c.seller_id,
        last_message_at: c.last_message_at,
        book_title: (c.books as any)?.title || "Knjiga",
        other_name: profileMap[otherId] || "Korisnik",
        last_message: lastMsg?.message_text || "",
        unread_count: count || 0,
      });
    }

    setConversations(previews);
    setLoading(false);
  };

  if (!user) return null;

  return (
    <div className="relative min-h-screen">
      <div className="fixed inset-0 -z-10 bg-cover bg-center bg-no-repeat" style={{ backgroundImage: `url(${chatBg})` }} />
      <div className="fixed inset-0 -z-10 bg-background/80 backdrop-blur-sm" />
      <Navbar />
      <div className="container mx-auto px-4 py-8 max-w-2xl">
        <h1 className="mb-6 font-display text-2xl font-bold text-foreground">Poruke</h1>

        {loading ? (
          <div className="flex justify-center py-20">
            <Loader2 className="h-8 w-8 animate-spin text-primary" />
          </div>
        ) : conversations.length === 0 ? (
          <div className="py-20 text-center">
            <MessageCircle className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
            <h3 className="mb-2 font-display text-lg font-semibold text-foreground">Nemate poruka</h3>
            <p className="text-sm text-muted-foreground">
              Kada pošaljete ili primite poruku, pojaviće se ovde.
            </p>
          </div>
        ) : (
          <div className="grid gap-2">
            {conversations.map((c) => (
              <Card
                key={c.id}
                className={`cursor-pointer transition-colors hover:bg-accent/50 ${c.unread_count > 0 ? "border-primary/40 bg-primary/5" : ""}`}
                onClick={() => navigate(`/conversation/${c.id}`)}
              >
                <CardContent className="flex items-center gap-3 p-4">
                  <div className="flex h-10 w-10 shrink-0 items-center justify-center rounded-full bg-primary/10 text-primary font-bold text-sm">
                    {c.other_name.charAt(0).toUpperCase()}
                  </div>
                  <div className="min-w-0 flex-1">
                    <div className="flex items-center justify-between gap-2">
                      <span className={`truncate text-sm ${c.unread_count > 0 ? "font-bold text-foreground" : "font-medium text-foreground"}`}>
                        {c.other_name}
                      </span>
                      <span className="shrink-0 text-xs text-muted-foreground">
                        {formatDistanceToNow(new Date(c.last_message_at), { addSuffix: true, locale: sr })}
                      </span>
                    </div>
                    <p className="truncate text-xs text-muted-foreground">{c.book_title}</p>
                    <p className={`mt-0.5 truncate text-sm ${c.unread_count > 0 ? "font-medium text-foreground" : "text-muted-foreground"}`}>
                      {c.last_message}
                    </p>
                  </div>
                  {c.unread_count > 0 && (
                    <Badge className="shrink-0 rounded-full px-2 py-0.5 text-xs">{c.unread_count}</Badge>
                  )}
                </CardContent>
              </Card>
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
import { useState, useEffect } from "react";
import { motion } from "framer-motion";
import { BookOpen, GraduationCap, ShoppingBag, Tag, ArrowRight } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Navbar } from "@/components/Navbar";
import { BookCard } from "@/components/BookCard";
import { supabase } from "@/integrations/supabase/client";
import { Tables } from "@/integrations/supabase/types";
import { useNavigate } from "react-router-dom";
import { useAuth } from "@/lib/auth-context";
import { useSavedBooks } from "@/hooks/useSavedBooks";
import heroImage from "@/assets/hero-books.jpg";

export default function Index() {
  const [latestBooks, setLatestBooks] = useState<Tables<"books">[]>([]);
  const [loading, setLoading] = useState(true);
  const navigate = useNavigate();
  const { user } = useAuth();
  const { savedIds, toggleSave } = useSavedBooks();

  useEffect(() => {
    const fetchLatest = async () => {
      const { data } = await supabase
        .from("books")
        .select("*")
        .eq("is_sold", false)
        .order("created_at", { ascending: false })
        .limit(8);
      setLatestBooks(data || []);
      setLoading(false);
    };
    fetchLatest();
  }, []);

  const handleCTA = (intent: "buy" | "sell") => {
    if (intent === "buy") {
      navigate("/shop"); // Anyone can browse books
    } else {
      // Selling requires authentication
      if (user) {
        navigate("/sell");
      } else {
        navigate("/auth?intent=sell");
      }
    }
  };

  return (
    <div className="min-h-screen bg-background">
      <Navbar />

      {/* Hero */}
      <section className="relative overflow-hidden gradient-hero py-24 md:py-32">
        <div className="absolute inset-0 opacity-20">
          <img src={heroImage} alt="" className="h-full w-full object-cover" />
        </div>
        <div className="container relative mx-auto px-4 text-center">
          <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 0.6 }}>
            <div className="mx-auto mb-6 flex h-16 w-16 items-center justify-center rounded-2xl gradient-primary shadow-primary">
              <BookOpen className="h-8 w-8 text-primary-foreground" />
            </div>
            <h1 className="heading-display mb-4 text-primary-foreground md:text-5xl lg:text-6xl">
              BookNook
            </h1>
            <p className="mx-auto mb-10 max-w-2xl text-lg text-primary-foreground/80">
              Marketplace za studentsku literaturu. Pronađi ili prodaj udžbenike brzo i jednostavno.
            </p>
            <div className="flex flex-col sm:flex-row justify-center gap-4">
              <Button
                size="lg"
                onClick={() => handleCTA("buy")}
                className="gradient-primary border-0 shadow-primary text-base px-8 py-6"
              >
                <ShoppingBag className="h-5 w-5 mr-2" />
                Kupi knjigu
              </Button>
              <Button
                size="lg"
                onClick={() => handleCTA("sell")}
                className="bg-primary/20 border border-primary-foreground/20 text-primary-foreground hover:bg-primary/40 text-base px-8 py-6 backdrop-blur-sm"
              >
                <Tag className="h-5 w-5 mr-2" />
                Prodaj knjigu
              </Button>
            </div>
            <div className="mt-6">
              <Button
                variant="ghost"
                size="sm"
                onClick={() => navigate("/chatbot")}
                className="text-primary-foreground/70 hover:text-primary-foreground hover:bg-primary-foreground/10"
              >
                AI Preporuke →
              </Button>
            </div>
          </motion.div>
        </div>
      </section>

      {/* Stats */}
      <section className="border-b border-border bg-card py-8">
        <div className="container mx-auto flex items-center justify-center px-4">
          <div className="flex items-center gap-3 rounded-xl bg-secondary/50 px-8 py-4">
            <div className="flex h-10 w-10 items-center justify-center rounded-lg gradient-primary">
              <GraduationCap className="h-5 w-5 text-primary-foreground" />
            </div>
            <div>
              <p className="font-display text-2xl font-bold text-foreground">12+</p>
              <p className="text-sm text-muted-foreground">Fakulteta širom Srbije</p>
            </div>
          </div>
        </div>
      </section>

      {/* Latest Books */}
      <section className="container mx-auto px-4 py-14">
        <div className="flex items-center justify-between mb-8">
          <div>
            <h2 className="heading-medium text-foreground">Najnovije knjige</h2>
            <p className="text-sm text-muted-foreground mt-2">Poslednje dodati udžbenici na platformi</p>
          </div>
          <Button variant="outline" onClick={() => navigate("/shop")} className="hidden sm:flex">
            Sve knjige <ArrowRight className="h-4 w-4 ml-2" />
          </Button>
        </div>

        {loading ? (
          <div className="grid gap-5 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
            {[...Array(8)].map((_, i) => (
              <div key={i} className="animate-pulse rounded-xl bg-muted">
                <div className="aspect-[3/4] rounded-t-xl bg-muted-foreground/10" />
                <div className="p-4 space-y-2">
                  <div className="h-4 w-3/4 rounded bg-muted-foreground/10" />
                  <div className="h-3 w-1/2 rounded bg-muted-foreground/10" />
                  <div className="h-6 w-1/3 rounded bg-muted-foreground/10 mt-3" />
                </div>
              </div>
            ))}
          </div>
        ) : latestBooks.length === 0 ? (
          <div className="py-16 text-center">
            <BookOpen className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
            <h3 className="mb-2 font-display text-lg font-semibold text-foreground">Još nema knjiga</h3>
            <p className="text-sm text-muted-foreground">Budite prvi koji će dodati udžbenik</p>
            <Button className="mt-4" onClick={() => handleCTA("sell")}>Dodaj prvu knjigu</Button>
          </div>
        ) : (
          <div className="grid gap-5 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
            {latestBooks.map((book) => (
              <BookCard key={book.id} book={book} isSaved={savedIds.has(book.id)} onToggleSave={toggleSave} />
            ))}
          </div>
        )}

        <div className="mt-8 text-center sm:hidden">
          <Button variant="outline" onClick={() => navigate("/shop")} className="w-full">
            Pregledaj sve knjige <ArrowRight className="h-4 w-4 ml-2" />
          </Button>
        </div>
      </section>
    </div>
  );
}
import { useState, useEffect } from "react";
import { useNavigate, Link } from "react-router-dom";
import { Loader2, Mail, ExternalLink } from "lucide-react";
import { Navbar } from "@/components/Navbar";
import { useAuth } from "@/lib/auth-context";
import { supabase } from "@/integrations/supabase/client";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { format } from "date-fns";

interface InquiryWithBook {
  id: string;
  buyer_name: string;
  buyer_email: string;
  message: string;
  created_at: string;
  book_id: string;
  books: { title: string; is_sold: boolean } | null;
}

export default function Inquiries() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const [inquiries, setInquiries] = useState<InquiryWithBook[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!user) {
      navigate("/auth");
      return;
    }
    fetchInquiries();
  }, [user]);

  const fetchInquiries = async () => {
    if (!user) return;
    setLoading(true);
    const { data } = await supabase
      .from("inquiries")
      .select("id, buyer_name, buyer_email, message, created_at, book_id, books(title, is_sold)")
      .eq("seller_id", user.id)
      .order("created_at", { ascending: false });
    setInquiries((data as unknown as InquiryWithBook[]) || []);
    setLoading(false);
  };

  if (!user) return null;

  return (
    <div className="min-h-screen bg-background">
      <Navbar />
      <div className="container mx-auto px-4 py-8">
        <div className="mb-8">
          <h1 className="font-display text-2xl font-bold text-foreground">Poruke</h1>
          <p className="text-sm text-muted-foreground">
            {inquiries.length} {inquiries.length === 1 ? "poruka" : "poruka"}
          </p>
        </div>

        {loading ? (
          <div className="flex justify-center py-20">
            <Loader2 className="h-8 w-8 animate-spin text-primary" />
          </div>
        ) : inquiries.length === 0 ? (
          <div className="py-20 text-center">
            <Mail className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
            <h3 className="mb-2 font-display text-lg font-semibold text-foreground">
              Nemate poruka
            </h3>
            <p className="text-sm text-muted-foreground">
              Kada neko pošalje upit za vašu knjigu, pojaviće se ovde.
            </p>
          </div>
        ) : (
          <div className="grid gap-4">
            {inquiries.map((inq) => (
              <Card key={inq.id}>
                <CardHeader className="pb-3">
                  <div className="flex items-start justify-between gap-2">
                    <div className="min-w-0">
                      <CardTitle className="text-base font-semibold">
                        {inq.books?.title || "Nepoznata knjiga"}
                      </CardTitle>
                      <p className="mt-1 text-xs text-muted-foreground">
                        {format(new Date(inq.created_at), "dd.MM.yyyy HH:mm")}
                      </p>
                    </div>
                    <div className="flex items-center gap-2 shrink-0">
                      {inq.books?.is_sold && (
                        <Badge variant="secondary">Prodato</Badge>
                      )}
                      <Link
                        to={`/listing/${inq.book_id}`}
                        className="text-primary hover:text-primary/80"
                      >
                        <ExternalLink className="h-4 w-4" />
                      </Link>
                    </div>
                  </div>
                </CardHeader>
                <CardContent className="pt-0">
                  <div className="mb-2 flex flex-wrap gap-x-4 gap-y-1 text-sm">
                    <span className="text-foreground font-medium">{inq.buyer_name}</span>
                    <a
                      href={`mailto:${inq.buyer_email}`}
                      className="text-primary hover:underline"
                    >
                      {inq.buyer_email}
                    </a>
                  </div>
                  <p className="text-sm text-muted-foreground whitespace-pre-wrap">{inq.message}</p>
                </CardContent>
              </Card>
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
import { useState, useEffect } from "react";
import { Search, Filter, X, FileText, Download, Calendar, User, Loader2, Upload } from "lucide-react";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { FacultySelect } from "@/components/FacultySelect";
import { Navbar } from "@/components/Navbar";
import { useAuth } from "@/lib/auth-context";
import { supabase } from "@/integrations/supabase/client";
import { useNavigate } from "react-router-dom";
import { format } from "date-fns";
import profileBg from "@/assets/profile-bg.jpg";

interface Material {
  id: string;
  user_id: string;
  title: string;
  faculty: string;
  subject: string;
  description: string | null;
  file_url: string;
  file_name: string;
  file_type: string;
  created_at: string;
  uploader_name?: string;
}

export default function Materials() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const [materials, setMaterials] = useState<Material[]>([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState("");
  const [faculty, setFaculty] = useState("");
  const [subjectFilter, setSubjectFilter] = useState("");

  useEffect(() => {
    fetchMaterials();
  }, []);

  const fetchMaterials = async () => {
    setLoading(true);
    const { data } = await supabase
      .from("materials")
      .select("*")
      .order("created_at", { ascending: false });

    if (data && data.length > 0) {
      const userIds = [...new Set(data.map((m: any) => m.user_id))];
      const { data: profiles } = await supabase
        .from("profiles")
        .select("user_id, full_name")
        .in("user_id", userIds);

      const profileMap = new Map(
        (profiles || []).map((p: any) => [p.user_id, p.full_name || "Korisnik"])
      );

      setMaterials(
        data.map((m: any) => ({
          ...m,
          uploader_name: profileMap.get(m.user_id) || "Korisnik",
        }))
      );
    } else {
      setMaterials([]);
    }
    setLoading(false);
  };

  const filtered = materials.filter((m) => {
    const q = search.toLowerCase();
    const matchSearch =
      !q ||
      m.title.toLowerCase().includes(q) ||
      m.subject.toLowerCase().includes(q) ||
      (m.description || "").toLowerCase().includes(q);
    const matchFaculty = !faculty || m.faculty === faculty;
    const matchSubject = !subjectFilter || m.subject.toLowerCase().includes(subjectFilter.toLowerCase());
    return matchSearch && matchFaculty && matchSubject;
  });

  const hasFilters = faculty || subjectFilter;

  const getFileIcon = (type: string) => {
    if (type.includes("pdf")) return "PDF";
    if (type.includes("presentation") || type.includes("ppt")) return "PPT";
    if (type.includes("word") || type.includes("doc")) return "DOC";
    return "FILE";
  };

  return (
    <div className="relative min-h-screen">
      <div className="fixed inset-0 -z-10">
        <img src={profileBg} alt="" className="h-full w-full object-cover" />
        <div className="absolute inset-0 bg-background/85 backdrop-blur-sm" />
      </div>

      <Navbar />

      <div className="container mx-auto max-w-6xl px-4 py-8">
        <div className="mb-8 flex items-center justify-between">
          <div>
            <h1 className="font-display text-3xl font-bold text-foreground">Studijski materijali</h1>
            <p className="mt-1 text-muted-foreground">Besplatni materijali za učenje - skripte, prezentacije i beleške</p>
          </div>
          {user && (
            <Button onClick={() => navigate("/upload-material")}>
              <Upload className="mr-1.5 h-4 w-4" />
              Dodaj materijal
            </Button>
          )}
        </div>

        {/* Filters */}
        <div className="mb-6 space-y-4">
          <div className="relative">
            <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-primary-dark" />
            <Input
              placeholder="Pretraži po naslovu, predmetu ili opisu..."
              value={search}
              onChange={(e) => setSearch(e.target.value)}
              className="pl-10 h-12 text-sm bg-card border-[hsl(var(--primary-dark))] text-primary-dark placeholder:text-[hsl(var(--primary-dark))]/60 focus-visible:ring-2 focus-visible:ring-[hsl(var(--primary))] focus-visible:border-[hsl(var(--primary))] transition-colors"
            />
          </div>
          <div className="flex flex-wrap gap-3 items-center">
            <Filter className="h-4 w-4 text-primary-dark" />
            <FacultySelect
              value={faculty}
              onValueChange={setFaculty}
              placeholder="Fakultet"
              className="w-[200px] bg-card border-[hsl(var(--primary-dark))] text-primary-dark"
            />
            <Input
              placeholder="Predmet..."
              value={subjectFilter}
              onChange={(e) => setSubjectFilter(e.target.value)}
              className="w-[200px] bg-card border-[hsl(var(--primary-dark))] text-primary-dark"
            />
            {hasFilters && (
              <Button variant="ghost" size="sm" onClick={() => { setFaculty(""); setSubjectFilter(""); }}>
                <X className="h-3 w-3 mr-1" /> Obriši filtere
              </Button>
            )}
          </div>
        </div>

        {/* Materials Grid */}
        {loading ? (
          <div className="flex justify-center py-20">
            <Loader2 className="h-8 w-8 animate-spin text-primary" />
          </div>
        ) : filtered.length === 0 ? (
          <div className="py-20 text-center">
            <FileText className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
            <h3 className="mb-2 font-display text-lg font-semibold text-foreground">
              Nema pronađenih materijala
            </h3>
            <p className="text-sm text-muted-foreground">
              {materials.length === 0
                ? "Budite prvi koji će podeliti studijske materijale!"
                : "Pokušajte sa drugim filterima"}
            </p>
          </div>
        ) : (
          <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
            {filtered.map((m) => (
              <Card key={m.id} className="overflow-hidden border-border bg-card hover:shadow-lg transition-shadow">
                <CardContent className="p-5">
                  <div className="mb-3 flex items-start justify-between">
                    <Badge variant="secondary" className="text-xs font-medium">
                      {getFileIcon(m.file_type)}
                    </Badge>
                    <span className="text-xs text-muted-foreground">
                      {format(new Date(m.created_at), "dd.MM.yyyy")}
                    </span>
                  </div>
                  <h3 className="mb-1 font-display text-base font-semibold text-foreground line-clamp-2">
                    {m.title}
                  </h3>
                  <p className="mb-2 text-sm text-primary-dark font-medium">{m.subject}</p>
                  <p className="mb-1 text-xs text-muted-foreground">{m.faculty}</p>
                  {m.description && (
                    <p className="mb-3 text-xs text-muted-foreground line-clamp-2">{m.description}</p>
                  )}
                  <div className="mt-3 flex items-center justify-between border-t border-border pt-3">
                    <div className="flex items-center gap-1 text-xs text-muted-foreground">
                      <User className="h-3 w-3" />
                      {m.uploader_name}
                    </div>
                    <Button size="sm" variant="outline" asChild>
                      <a href={m.file_url} target="_blank" rel="noopener noreferrer" download>
                        <Download className="mr-1 h-3.5 w-3.5" />
                        Preuzmi
                      </a>
                    </Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
import { useLocation } from "react-router-dom";
import { useEffect } from "react";

const NotFound = () => {
  const location = useLocation();

  useEffect(() => {
    console.error("404 Error: User attempted to access non-existent route:", location.pathname);
  }, [location.pathname]);

  return (
    <div className="flex min-h-screen items-center justify-center bg-muted">
      <div className="text-center">
        <h1 className="mb-4 text-4xl font-bold">404</h1>
        <p className="mb-4 text-xl text-muted-foreground">Oops! Page not found</p>
        <a href="/" className="text-primary underline hover:text-primary/90">
          Return to Home
        </a>
      </div>
    </div>
  );
};

export default NotFound;
import { useState, useEffect } from "react";
import { useNavigate } from "react-router-dom";
import { Trash2, BookOpen, Plus, Loader2, Pencil, Heart, MessageCircle, FileText, Download } from "lucide-react";
import profileBg from "@/assets/profile-bg.jpg";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Navbar } from "@/components/Navbar";
import { BookCard } from "@/components/BookCard";
import { ProfileHeader } from "@/components/profile/ProfileHeader";
import { ProfileStats } from "@/components/profile/ProfileStats";
import { EditProfileDialog } from "@/components/profile/EditProfileDialog";
import { useAuth } from "@/lib/auth-context";
import { supabase } from "@/integrations/supabase/client";
import { Tables } from "@/integrations/supabase/types";
import { toast } from "sonner";
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from "@/components/ui/alert-dialog";

export default function Profile() {
  const { user } = useAuth();
  const navigate = useNavigate();

  const [profile, setProfile] = useState<Tables<"profiles"> | null>(null);
  const [books, setBooks] = useState<Tables<"books">[]>([]);
  const [savedBooks, setSavedBooks] = useState<Tables<"books">[]>([]);
  const [loading, setLoading] = useState(true);
  const [savedLoading, setSavedLoading] = useState(true);
  const [deletingId, setDeletingId] = useState<string | null>(null);
  const [editOpen, setEditOpen] = useState(false);
  const [soldCount, setSoldCount] = useState(0);
  const [myMaterials, setMyMaterials] = useState<any[]>([]);
  const [materialsLoading, setMaterialsLoading] = useState(true);

  useEffect(() => {
    if (!user) {
      navigate("/auth");
      return;
    }
    fetchProfile();
    fetchMyBooks();
    fetchSavedBooks();
    fetchMyMaterials();
  }, [user]);

  const fetchProfile = async () => {
    if (!user) return;
    const { data } = await supabase
      .from("profiles")
      .select("*")
      .eq("user_id", user.id)
      .single();
    if (data) setProfile(data);
  };

  const fetchMyBooks = async () => {
    if (!user) return;
    setLoading(true);
    const { data } = await supabase
      .from("books")
      .select("*")
      .eq("seller_id", user.id)
      .order("created_at", { ascending: false });
    const allBooks = data || [];
    setBooks(allBooks);
    setSoldCount(allBooks.filter((b) => b.is_sold).length);
    setLoading(false);
  };

  const fetchSavedBooks = async () => {
    if (!user) return;
    setSavedLoading(true);
    const { data: saved } = await supabase
      .from("saved_books")
      .select("book_id")
      .eq("user_id", user.id);
    if (saved && saved.length > 0) {
      const ids = saved.map((s: any) => s.book_id);
      const { data: booksData } = await supabase
        .from("books")
        .select("*")
        .in("id", ids)
        .order("created_at", { ascending: false });
      setSavedBooks(booksData || []);
    } else {
      setSavedBooks([]);
    }
    setSavedLoading(false);
  };

  const fetchMyMaterials = async () => {
    if (!user) return;
    setMaterialsLoading(true);
    const { data } = await supabase
      .from("materials")
      .select("*")
      .eq("user_id", user.id)
      .order("created_at", { ascending: false });
    setMyMaterials(data || []);
    setMaterialsLoading(false);
  };

  const handleDeleteMaterial = async (id: string) => {
    if (!user) return;
    const material = myMaterials.find((m) => m.id === id);
    if (material) {
      const urlParts = material.file_url.split("/study-materials/");
      if (urlParts[1]) {
        await supabase.storage.from("study-materials").remove([decodeURIComponent(urlParts[1])]);
      }
    }
    await supabase.from("materials").delete().eq("id", id);
    setMyMaterials((prev) => prev.filter((m) => m.id !== id));
    toast.success("Materijal je obrisan");
  };

  const handleDelete = async (bookId: string) => {
    if (!user) return;
    setDeletingId(bookId);
    try {
      const { data: files } = await supabase.storage
        .from("book-images")
        .list(`${user.id}/${bookId}`);
      if (files && files.length > 0) {
        const paths = files.map((f) => `${user.id}/${bookId}/${f.name}`);
        await supabase.storage.from("book-images").remove(paths);
      }
      const { error } = await supabase.from("books").delete().eq("id", bookId);
      if (error) throw error;
      setBooks((prev) => prev.filter((b) => b.id !== bookId));
      toast.success("Knjiga je obrisana");
    } catch (err) {
      console.error(err);
      toast.error("Greška pri brisanju knjige");
    } finally {
      setDeletingId(null);
    }
  };

  const handleUnsave = async (bookId: string) => {
    if (!user) return;
    await supabase.from("saved_books").delete().eq("user_id", user.id).eq("book_id", bookId);
    setSavedBooks((prev) => prev.filter((b) => b.id !== bookId));
    toast.success("Knjiga uklonjena iz sačuvanih");
  };

  const handleProfileSave = (data: { fullName: string; faculty: string }) => {
    setProfile((prev) => prev ? { ...prev, full_name: data.fullName, faculty: data.faculty } : prev);
  };

  const activeListings = books.filter((b) => !b.is_sold).length;

  if (!user) return null;

  return (
    <div className="relative min-h-screen">
      {/* Background */}
      <div className="fixed inset-0 -z-10">
        <img src={profileBg} alt="" className="h-full w-full object-cover" />
        <div className="absolute inset-0 bg-background/85 backdrop-blur-sm" />
      </div>

      <Navbar />
      <div className="container mx-auto max-w-4xl px-4 py-8">
        {/* Profile Header Card */}
        {profile ? (
          <ProfileHeader
            userId={user.id}
            email={user.email || ""}
            fullName={profile.full_name}
            faculty={profile.faculty}
            avatarUrl={profile.avatar_url}
            onAvatarChange={(url) => setProfile((p) => p ? { ...p, avatar_url: url } : p)}
            onEditClick={() => setEditOpen(true)}
          />
        ) : (
          <div className="flex justify-center rounded-2xl border border-border bg-card p-12">
            <Loader2 className="h-8 w-8 animate-spin text-primary" />
          </div>
        )}

        {/* Stats */}
        <div className="mt-6">
          <ProfileStats
            activeListings={activeListings}
            soldBooks={soldCount}
            savedBooks={savedBooks.length}
          />
        </div>

        {/* Quick action */}
        <div className="mt-6 flex justify-end">
          <Button onClick={() => navigate("/sell")}>
            <Plus className="mr-1 h-4 w-4" />
            Dodaj knjigu
          </Button>
        </div>

        {/* Tabs */}
        <Tabs defaultValue="listings" className="mt-6 w-full">
          <TabsList className="mb-6 w-full justify-start">
            <TabsTrigger value="listings">
              <BookOpen className="mr-1.5 h-4 w-4" />
              Oglasi ({books.length})
            </TabsTrigger>
            <TabsTrigger value="saved">
              <Heart className="mr-1.5 h-4 w-4" />
              Sačuvano ({savedBooks.length})
            </TabsTrigger>
            <TabsTrigger value="materials">
              <FileText className="mr-1.5 h-4 w-4" />
              Materijali ({myMaterials.length})
            </TabsTrigger>
            <TabsTrigger value="messages">
              <MessageCircle className="mr-1.5 h-4 w-4" />
              Poruke
            </TabsTrigger>
            <TabsTrigger value="support">
              Podrška
            </TabsTrigger>
          </TabsList>

          {/* Listings */}
          <TabsContent value="listings">
            {loading ? (
              <div className="flex justify-center py-20">
                <Loader2 className="h-8 w-8 animate-spin text-primary" />
              </div>
            ) : books.length === 0 ? (
              <div className="py-20 text-center">
                <BookOpen className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
                <h3 className="mb-2 font-display text-lg font-semibold text-foreground">
                  Nemate objavljenih oglasa
                </h3>
                <p className="mb-4 text-sm text-muted-foreground">
                  Dodajte svoju prvu knjigu i počnite sa prodajom
                </p>
                <Button onClick={() => navigate("/sell")}>Dodaj prvu knjigu</Button>
              </div>
            ) : (
              <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
                {books.map((book) => (
                  <div key={book.id} className="relative">
                    <BookCard book={book} />
                    <div className="absolute right-2 top-2 z-10 flex gap-1">
                      <Button
                        variant="secondary"
                        size="icon"
                        className="h-8 w-8 shadow-md"
                        onClick={() => navigate(`/edit/${book.id}`)}
                      >
                        <Pencil className="h-4 w-4" />
                      </Button>
                      <AlertDialog>
                        <AlertDialogTrigger asChild>
                          <Button
                            variant="destructive"
                            size="icon"
                            className="h-8 w-8 shadow-md"
                            disabled={deletingId === book.id}
                          >
                            {deletingId === book.id ? (
                              <Loader2 className="h-4 w-4 animate-spin" />
                            ) : (
                              <Trash2 className="h-4 w-4" />
                            )}
                          </Button>
                        </AlertDialogTrigger>
                        <AlertDialogContent>
                          <AlertDialogHeader>
                            <AlertDialogTitle>Obrisati oglas?</AlertDialogTitle>
                            <AlertDialogDescription>
                              Da li ste sigurni da želite da obrišete &quot;{book.title}&quot;? Ova akcija se ne može poništiti.
                            </AlertDialogDescription>
                          </AlertDialogHeader>
                          <AlertDialogFooter>
                            <AlertDialogCancel>Otkaži</AlertDialogCancel>
                            <AlertDialogAction onClick={() => handleDelete(book.id)}>
                              Obriši
                            </AlertDialogAction>
                          </AlertDialogFooter>
                        </AlertDialogContent>
                      </AlertDialog>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </TabsContent>

          {/* Saved */}
          <TabsContent value="saved">
            {savedLoading ? (
              <div className="flex justify-center py-20">
                <Loader2 className="h-8 w-8 animate-spin text-primary" />
              </div>
            ) : savedBooks.length === 0 ? (
              <div className="py-20 text-center">
                <Heart className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
                <h3 className="mb-2 font-display text-lg font-semibold text-foreground">
                  Nemate sačuvanih knjiga
                </h3>
                <p className="mb-4 text-sm text-muted-foreground">
                  Kliknite na srce na bilo kojoj knjizi da je sačuvate
                </p>
                <Button variant="outline" onClick={() => navigate("/shop")}>Pretraži knjige</Button>
              </div>
            ) : (
              <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
                {savedBooks.map((book) => (
                  <div key={book.id} className="relative">
                    <BookCard book={book} />
                    <div className="absolute right-2 top-2 z-10">
                      <Button
                        variant="secondary"
                        size="icon"
                        className="h-8 w-8 shadow-md"
                        onClick={() => handleUnsave(book.id)}
                      >
                        <Heart className="h-4 w-4 fill-destructive text-destructive" />
                      </Button>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </TabsContent>

          {/* Materials */}
          <TabsContent value="materials">
            {materialsLoading ? (
              <div className="flex justify-center py-20">
                <Loader2 className="h-8 w-8 animate-spin text-primary" />
              </div>
            ) : myMaterials.length === 0 ? (
              <div className="py-20 text-center">
                <FileText className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
                <h3 className="mb-2 font-display text-lg font-semibold text-foreground">
                  Nemate uploadovanih materijala
                </h3>
                <p className="mb-4 text-sm text-muted-foreground">
                  Podelite skripte, prezentacije ili beleške sa kolegama
                </p>
                <Button onClick={() => navigate("/upload-material")}>Dodaj materijal</Button>
              </div>
            ) : (
              <div className="space-y-3">
                <div className="flex justify-end">
                  <Button size="sm" onClick={() => navigate("/upload-material")}>
                    <FileText className="mr-1 h-4 w-4" /> Dodaj materijal
                  </Button>
                </div>
                <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
                  {myMaterials.map((m) => (
                    <Card key={m.id} className="border-border bg-card">
                      <CardContent className="p-4">
                        <h4 className="font-display text-sm font-semibold text-foreground line-clamp-1">{m.title}</h4>
                        <p className="text-xs text-muted-foreground">{m.subject} • {m.faculty}</p>
                        <div className="mt-3 flex gap-2">
                          <Button size="sm" variant="outline" asChild className="flex-1">
                            <a href={m.file_url} target="_blank" rel="noopener noreferrer">
                              <Download className="mr-1 h-3 w-3" /> Preuzmi
                            </a>
                          </Button>
                          <AlertDialog>
                            <AlertDialogTrigger asChild>
                              <Button size="sm" variant="destructive">
                                <Trash2 className="h-3 w-3" />
                              </Button>
                            </AlertDialogTrigger>
                            <AlertDialogContent>
                              <AlertDialogHeader>
                                <AlertDialogTitle>Obrisati materijal?</AlertDialogTitle>
                                <AlertDialogDescription>
                                  Da li ste sigurni da želite da obrišete &quot;{m.title}&quot;?
                                </AlertDialogDescription>
                              </AlertDialogHeader>
                              <AlertDialogFooter>
                                <AlertDialogCancel>Otkaži</AlertDialogCancel>
                                <AlertDialogAction onClick={() => handleDeleteMaterial(m.id)}>
                                  Obriši
                                </AlertDialogAction>
                              </AlertDialogFooter>
                            </AlertDialogContent>
                          </AlertDialog>
                        </div>
                      </CardContent>
                    </Card>
                  ))}
                </div>
              </div>
            )}
          </TabsContent>

          {/* Messages shortcut */}
          <TabsContent value="messages">
            <div className="py-20 text-center">
              <MessageCircle className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
              <h3 className="mb-2 font-display text-lg font-semibold text-foreground">
                Vaše poruke
              </h3>
              <p className="mb-4 text-sm text-muted-foreground">
                Pregledajte sve vaše razgovore na jednom mjestu
              </p>
              <Button onClick={() => navigate("/inbox")}>Otvori inbox</Button>
            </div>
          </TabsContent>

          {/* Support shortcut */}
          <TabsContent value="support">
            <div className="py-20 text-center">
              <MessageCircle className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
              <h3 className="mb-2 font-display text-lg font-semibold text-foreground">
                Podrška
              </h3>
              <p className="mb-4 text-sm text-muted-foreground">
                Pregledajte vaše upite i razgovore sa BookNook timom
              </p>
              <Button onClick={() => navigate("/support")}>Moji upiti</Button>
            </div>
          </TabsContent>
        </Tabs>

        {/* Edit dialog */}
        {profile && (
          <EditProfileDialog
            open={editOpen}
            onOpenChange={setEditOpen}
            userId={user.id}
            fullName={profile.full_name}
            faculty={profile.faculty}
            onSave={handleProfileSave}
          />
        )}
      </div>
    </div>
  );
}
import { useState } from "react";
import { useNavigate } from "react-router-dom";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Card, CardContent, CardHeader } from "@/components/ui/card";
import { Navbar } from "@/components/Navbar";
import { ImageUpload } from "@/components/ImageUpload";
import { FacultySelect } from "@/components/FacultySelect";
import { useAuth } from "@/lib/auth-context";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";


export default function SellBook() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const [loading, setLoading] = useState(false);
  const [images, setImages] = useState<File[]>([]);
  const [previews, setPreviews] = useState<string[]>([]);
  const [form, setForm] = useState({
    title: "", author: "", faculty: "", subject: "",
    year_of_study: "", price: "", condition: "", description: "",
  });

  const update = (key: string, value: string) => setForm((p) => ({ ...p, [key]: value }));

  const handleAddImages = (files: File[]) => {
    const newImages = [...images, ...files];
    setImages(newImages);
    const newPreviews = files.map((f) => URL.createObjectURL(f));
    setPreviews((p) => [...p, ...newPreviews]);
  };

  const handleRemoveImage = (index: number) => {
    URL.revokeObjectURL(previews[index]);
    setImages((prev) => prev.filter((_, i) => i !== index));
    setPreviews((prev) => prev.filter((_, i) => i !== index));
  };

  const uploadImages = async (bookId: string): Promise<string | null> => {
    if (images.length === 0) return null;

    const uploadPromises = images.map(async (file, i) => {
      const ext = file.name.split(".").pop();
      const path = `${user!.id}/${bookId}/${i}.${ext}`;
      const { error } = await supabase.storage.from("book-images").upload(path, file, { upsert: true });
      if (error) throw error;
      const { data } = supabase.storage.from("book-images").getPublicUrl(path);
      return data.publicUrl;
    });

    const urls = await Promise.all(uploadPromises);
    return urls[0] || null;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!user) { toast.error("Morate biti prijavljeni"); return; }
    setLoading(true);

    try {
      // Insert book first to get ID
      const { data: book, error } = await supabase.from("books").insert({
        seller_id: user.id,
        title: form.title,
        author: form.author,
        faculty: form.faculty,
        subject: form.subject,
        year_of_study: parseInt(form.year_of_study),
        price: parseFloat(form.price),
        condition: form.condition,
        description: form.description || null,
      }).select("id").single();

      if (error) throw error;

      // Upload images and update book with primary image URL
      const imageUrl = await uploadImages(book.id);
      if (imageUrl) {
        await supabase.from("books").update({ image_url: imageUrl }).eq("id", book.id);
      }

      toast.success("Knjiga je uspešno dodata!");
      navigate("/");
    } catch (err) {
      toast.error("Greška pri dodavanju knjige");
      console.error(err);
    } finally {
      setLoading(false);
    }
  };

  if (!user) {
    navigate("/auth");
    return null;
  }

  return (
    <div className="min-h-screen bg-background">
      <Navbar />
      <div className="container mx-auto max-w-2xl px-4 py-8">
        <Card className="shadow-card">
          <CardHeader>
            <h1 className="font-display text-2xl font-bold text-foreground">Prodaj knjigu</h1>
            <p className="text-sm text-muted-foreground">Popunite informacije o knjizi koju želite da prodate</p>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleSubmit} className="space-y-4">
              {/* Image Upload */}
              <div className="space-y-2">
                <Label>Fotografije knjige</Label>
                <ImageUpload
                  images={images}
                  previews={previews}
                  onAdd={handleAddImages}
                  onRemove={handleRemoveImage}
                  maxImages={5}
                  disabled={loading}
                />
              </div>

              <div className="grid gap-4 sm:grid-cols-2">
                <div className="space-y-2">
                  <Label>Naslov knjige *</Label>
                  <Input value={form.title} onChange={(e) => update("title", e.target.value)} required />
                </div>
                <div className="space-y-2">
                  <Label>Autor *</Label>
                  <Input value={form.author} onChange={(e) => update("author", e.target.value)} required />
                </div>
              </div>
              <div className="grid gap-4 sm:grid-cols-2">
                <div className="space-y-2">
                  <Label>Fakultet *</Label>
                  <FacultySelect value={form.faculty} onValueChange={(v) => update("faculty", v)} disabled={loading} />
                </div>
                <div className="space-y-2">
                  <Label>Predmet *</Label>
                  <Input value={form.subject} onChange={(e) => update("subject", e.target.value)} required />
                </div>
              </div>
              <div className="grid gap-4 sm:grid-cols-3">
                <div className="space-y-2">
                  <Label>Godina studija *</Label>
                  <Select value={form.year_of_study} onValueChange={(v) => update("year_of_study", v)}>
                    <SelectTrigger><SelectValue placeholder="Godina" /></SelectTrigger>
                    <SelectContent>
                      {[1,2,3,4,5,6].map((y) => <SelectItem key={y} value={String(y)}>{y}. godina</SelectItem>)}
                    </SelectContent>
                  </Select>
                </div>
                <div className="space-y-2">
                  <Label>Cena (RSD) *</Label>
                  <Input type="number" min="0" value={form.price} onChange={(e) => update("price", e.target.value)} required />
                </div>
                <div className="space-y-2">
                  <Label>Stanje *</Label>
                  <Select value={form.condition} onValueChange={(v) => update("condition", v)}>
                    <SelectTrigger><SelectValue placeholder="Stanje" /></SelectTrigger>
                    <SelectContent>
                      <SelectItem value="novo">Novo</SelectItem>
                      <SelectItem value="kao novo">Kao novo</SelectItem>
                      <SelectItem value="dobro">Dobro</SelectItem>
                      <SelectItem value="korisceno">Korišćeno</SelectItem>
                    </SelectContent>
                  </Select>
                </div>
              </div>
              <div className="space-y-2">
                <Label>Opis (opciono)</Label>
                <Textarea value={form.description} onChange={(e) => update("description", e.target.value)} rows={3} />
              </div>
              <Button type="submit" className="w-full" disabled={loading}>
                {loading ? "Dodavanje..." : "Dodaj knjigu"}
              </Button>
            </form>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
import { useState, useEffect, useMemo } from "react";
import { BookOpen, SlidersHorizontal, ArrowUpDown } from "lucide-react";
import shopBg from "@/assets/shop-bg.jpg";
import { Navbar } from "@/components/Navbar";
import { BookCard } from "@/components/BookCard";
import { SearchAndFilters } from "@/components/SearchAndFilters";
import { Button } from "@/components/ui/button";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { supabase } from "@/integrations/supabase/client";
import { Tables } from "@/integrations/supabase/types";
import { useSavedBooks } from "@/hooks/useSavedBooks";

const BOOKS_PER_PAGE = 12;

type SortOption = "newest" | "price_asc" | "price_desc";

export default function Shop() {
  const [books, setBooks] = useState<Tables<"books">[]>([]);
  const [loading, setLoading] = useState(true);
  const [filters, setFilters] = useState({ search: "", faculty: "", year: "", condition: "" });
  const [sort, setSort] = useState<SortOption>("newest");
  const [page, setPage] = useState(1);
  const { savedIds, toggleSave } = useSavedBooks();

  useEffect(() => {
    setPage(1);
  }, [filters, sort]);

  useEffect(() => {
    fetchBooks();
  }, [filters, sort]);

  const fetchBooks = async () => {
    setLoading(true);
    let query = supabase.from("books").select("*").eq("is_sold", false);

    if (filters.faculty) query = query.eq("faculty", filters.faculty);
    if (filters.year) query = query.eq("year_of_study", parseInt(filters.year));
    if (filters.condition) query = query.eq("condition", filters.condition);
    if (filters.search) {
      query = query.or(`title.ilike.%${filters.search}%,author.ilike.%${filters.search}%,subject.ilike.%${filters.search}%`);
    }

    switch (sort) {
      case "price_asc":
        query = query.order("price", { ascending: true });
        break;
      case "price_desc":
        query = query.order("price", { ascending: false });
        break;
      default:
        query = query.order("created_at", { ascending: false });
    }

    const { data } = await query;
    setBooks(data || []);
    setLoading(false);
  };

  const totalPages = Math.ceil(books.length / BOOKS_PER_PAGE);
  const paginatedBooks = useMemo(
    () => books.slice((page - 1) * BOOKS_PER_PAGE, page * BOOKS_PER_PAGE),
    [books, page]
  );

  const pageNumbers = useMemo(() => {
    const pages: number[] = [];
    const start = Math.max(1, page - 2);
    const end = Math.min(totalPages, page + 2);
    for (let i = start; i <= end; i++) pages.push(i);
    return pages;
  }, [page, totalPages]);

  return (
    <div className="min-h-screen bg-background">
      <Navbar />

      {/* Header with background image */}
      <div className="relative overflow-hidden border-b border-border">
        <div
          className="absolute inset-0 bg-cover bg-center"
          style={{ backgroundImage: `url(${shopBg})` }}
        />
        <div className="absolute inset-0 bg-gradient-to-b from-primary/70 via-primary/50 to-background" />
        <div className="relative container mx-auto px-4 py-14">
          <div className="flex items-center gap-3 mb-2">
            <div className="flex h-10 w-10 items-center justify-center rounded-xl bg-primary-foreground/20 backdrop-blur-sm">
              <SlidersHorizontal className="h-5 w-5 text-primary-foreground" />
            </div>
            <h1 className="font-display text-3xl font-bold text-primary-foreground">Prodavnica knjiga</h1>
          </div>
          <p className="text-primary-foreground/80 ml-[52px]">Pretražite i pronađite udžbenike koje tražite</p>
        </div>
      </div>

      <div className="container mx-auto px-4 py-8">
        <div className="rounded-xl border border-primary/20 bg-card/80 backdrop-blur-sm p-5 shadow-card">
          <SearchAndFilters filters={filters} onChange={setFilters} />
        </div>

        <div className="mt-8">
          {loading ? (
            <div className="grid gap-5 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
              {[...Array(BOOKS_PER_PAGE)].map((_, i) => (
                <div key={i} className="animate-pulse rounded-xl bg-muted">
                  <div className="aspect-[3/4] rounded-t-xl bg-muted-foreground/10" />
                  <div className="p-4 space-y-2">
                    <div className="h-4 w-3/4 rounded bg-muted-foreground/10" />
                    <div className="h-3 w-1/2 rounded bg-muted-foreground/10" />
                    <div className="h-6 w-1/3 rounded bg-muted-foreground/10 mt-3" />
                  </div>
                </div>
              ))}
            </div>
          ) : books.length === 0 ? (
            <div className="py-20 text-center">
              <BookOpen className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
              <h3 className="mb-2 font-display text-lg font-semibold text-foreground">Nema rezultata</h3>
              <p className="text-sm text-muted-foreground">Nismo pronašli knjige koje odgovaraju vašoj pretrazi. Pokušajte sa drugim predmetom ili fakultetom.</p>
            </div>
          ) : (
            <>
              <div className="flex items-center justify-between mb-5">
                <p className="text-sm text-muted-foreground">
                  Prikazano {(page - 1) * BOOKS_PER_PAGE + 1}–{Math.min(page * BOOKS_PER_PAGE, books.length)} od {books.length} knjiga
                </p>
                <div className="flex items-center gap-2">
                  <ArrowUpDown className="h-4 w-4 text-muted-foreground" />
                  <Select value={sort} onValueChange={(v) => setSort(v as SortOption)}>
                    <SelectTrigger className="w-[180px] h-9 text-sm">
                      <SelectValue />
                    </SelectTrigger>
                    <SelectContent>
                      <SelectItem value="newest">Najnovije</SelectItem>
                      <SelectItem value="price_asc">Cena: niska → visoka</SelectItem>
                      <SelectItem value="price_desc">Cena: visoka → niska</SelectItem>
                    </SelectContent>
                  </Select>
                </div>
              </div>
              <div className="grid gap-5 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
                {paginatedBooks.map((book) => (
                  <BookCard key={book.id} book={book} isSaved={savedIds.has(book.id)} onToggleSave={toggleSave} />
                ))}
              </div>

              {/* Pagination */}
              {totalPages > 1 && (
                <div className="mt-10 flex items-center justify-center gap-2">
                  <Button
                    variant="outline"
                    size="sm"
                    disabled={page === 1}
                    onClick={() => setPage((p) => p - 1)}
                  >
                    Prethodna
                  </Button>
                  {pageNumbers[0] > 1 && (
                    <>
                      <Button variant={page === 1 ? "default" : "ghost"} size="sm" onClick={() => setPage(1)}>1</Button>
                      {pageNumbers[0] > 2 && <span className="px-1 text-muted-foreground">…</span>}
                    </>
                  )}
                  {pageNumbers.map((n) => (
                    <Button
                      key={n}
                      variant={page === n ? "default" : "ghost"}
                      size="sm"
                      onClick={() => setPage(n)}
                      className={page === n ? "gradient-primary border-0 shadow-primary" : ""}
                    >
                      {n}
                    </Button>
                  ))}
                  {pageNumbers[pageNumbers.length - 1] < totalPages && (
                    <>
                      {pageNumbers[pageNumbers.length - 1] < totalPages - 1 && <span className="px-1 text-muted-foreground">…</span>}
                      <Button variant={page === totalPages ? "default" : "ghost"} size="sm" onClick={() => setPage(totalPages)}>{totalPages}</Button>
                    </>
                  )}
                  <Button
                    variant="outline"
                    size="sm"
                    disabled={page === totalPages}
                    onClick={() => setPage((p) => p + 1)}
                  >
                    Sledeća
                  </Button>
                </div>
              )}
            </>
          )}
        </div>
      </div>
    </div>
  );
}
import { useEffect, useState } from "react";
import { useNavigate } from "react-router-dom";
import { supabase } from "@/integrations/supabase/client";
import { useAuth } from "@/lib/auth-context";
import { Navbar } from "@/components/Navbar";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Loader2, MessageSquare, ChevronRight } from "lucide-react";
import { format } from "date-fns";

interface SupportConvo {
  id: string;
  subject: string;
  created_at: string;
  last_message_at: string;
}

export default function Support() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const [convos, setConvos] = useState<SupportConvo[]>([]);
  const [unreadMap, setUnreadMap] = useState<Record<string, number>>({});
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!user) { navigate("/auth"); return; }
    fetchConvos();
  }, [user]);

  const fetchConvos = async () => {
    const { data } = await supabase
      .from("support_conversations")
      .select("*")
      .eq("user_id", user!.id)
      .order("last_message_at", { ascending: false });
    const list = (data as SupportConvo[]) || [];
    setConvos(list);

    // Get unread counts
    if (list.length > 0) {
      const { data: msgs } = await supabase
        .from("support_messages")
        .select("conversation_id")
        .in("conversation_id", list.map((c) => c.id))
        .eq("is_read", false)
        .eq("is_admin", true);
      const map: Record<string, number> = {};
      (msgs || []).forEach((m: any) => {
        map[m.conversation_id] = (map[m.conversation_id] || 0) + 1;
      });
      setUnreadMap(map);
    }
    setLoading(false);
  };

  return (
    <div className="min-h-screen bg-background">
      <Navbar />
      <div className="container mx-auto max-w-3xl px-4 py-8">
        <h1 className="font-display text-2xl font-bold text-foreground mb-6">Moji upiti</h1>

        {loading ? (
          <div className="flex justify-center py-16"><Loader2 className="h-8 w-8 animate-spin text-primary" /></div>
        ) : convos.length === 0 ? (
          <div className="py-20 text-center">
            <MessageSquare className="mx-auto mb-4 h-12 w-12 text-muted-foreground/30" />
            <h3 className="mb-2 font-display text-lg font-semibold text-foreground">Nemate upita</h3>
            <p className="text-sm text-muted-foreground">Pošaljite poruku putem kontakt forme da započnete razgovor sa timom.</p>
          </div>
        ) : (
          <div className="space-y-3">
            {convos.map((c) => (
              <Card
                key={c.id}
                className="cursor-pointer border-border hover:border-primary/30 transition-colors"
                onClick={() => navigate(`/support/${c.id}`)}
              >
                <CardContent className="flex items-center justify-between p-4">
                  <div className="flex-1 min-w-0">
                    <div className="flex items-center gap-2">
                      <h3 className="font-medium text-foreground truncate">{c.subject}</h3>
                      {unreadMap[c.id] && (
                        <Badge className="shrink-0">{unreadMap[c.id]}</Badge>
                      )}
                    </div>
                    <p className="text-xs text-muted-foreground mt-0.5">
                      {format(new Date(c.last_message_at), "dd.MM.yyyy HH:mm")}
                    </p>
                  </div>
                  <ChevronRight className="h-5 w-5 text-muted-foreground shrink-0" />
                </CardContent>
              </Card>
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
import { useEffect, useState, useRef } from "react";
import { useParams, useNavigate } from "react-router-dom";
import { supabase } from "@/integrations/supabase/client";
import { useAuth } from "@/lib/auth-context";
import { useAdmin } from "@/hooks/useAdmin";
import { Navbar } from "@/components/Navbar";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Loader2, ArrowLeft, Send, Shield, User } from "lucide-react";
import { format } from "date-fns";

interface SupportMessage {
  id: string;
  conversation_id: string;
  sender_id: string;
  message_text: string;
  is_admin: boolean;
  is_read: boolean;
  created_at: string;
}

interface SupportConvo {
  id: string;
  user_id: string;
  subject: string;
  created_at: string;
}

export default function SupportConversation() {
  const { id } = useParams<{ id: string }>();
  const { user } = useAuth();
  const { isAdmin } = useAdmin();
  const navigate = useNavigate();
  const [convo, setConvo] = useState<SupportConvo | null>(null);
  const [messages, setMessages] = useState<SupportMessage[]>([]);
  const [loading, setLoading] = useState(true);
  const [reply, setReply] = useState("");
  const [sending, setSending] = useState(false);
  const bottomRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!user || !id) return;
    fetchData();

    const channel = supabase
      .channel(`support-${id}`)
      .on("postgres_changes", { event: "INSERT", schema: "public", table: "support_messages", filter: `conversation_id=eq.${id}` }, (payload) => {
        setMessages((prev) => [...prev, payload.new as SupportMessage]);
      })
      .subscribe();

    return () => { supabase.removeChannel(channel); };
  }, [user, id]);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  // Mark unread messages as read
  useEffect(() => {
    if (!user || !messages.length) return;
    const unread = messages.filter((m) => !m.is_read && m.sender_id !== user.id);
    if (unread.length > 0) {
      supabase
        .from("support_messages")
        .update({ is_read: true } as any)
        .in("id", unread.map((m) => m.id))
        .then();
    }
  }, [messages, user]);

  const fetchData = async () => {
    const { data: c } = await supabase
      .from("support_conversations")
      .select("*")
      .eq("id", id!)
      .single();
    setConvo(c as SupportConvo | null);

    const { data: msgs } = await supabase
      .from("support_messages")
      .select("*")
      .eq("conversation_id", id!)
      .order("created_at", { ascending: true });
    setMessages((msgs as SupportMessage[]) || []);
    setLoading(false);
  };

  const handleSend = async () => {
    if (!reply.trim() || !user || !id) return;
    setSending(true);
    await supabase.from("support_messages").insert([{
      conversation_id: id,
      sender_id: user.id,
      message_text: reply.trim(),
      is_admin: isAdmin,
    } as any]);
    await supabase
      .from("support_conversations")
      .update({ last_message_at: new Date().toISOString() } as any)
      .eq("id", id);
    setReply("");
    setSending(false);
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault();
      handleSend();
    }
  };

  if (!user) return null;

  const backPath = isAdmin ? "/admin/support" : "/support";

  return (
    <div className="flex min-h-screen flex-col bg-background">
      <Navbar />
      <div className="container mx-auto flex max-w-3xl flex-1 flex-col px-4 py-4">
        {/* Header */}
        <div className="mb-4 flex items-center gap-3">
          <Button variant="ghost" size="icon" onClick={() => navigate(backPath)}>
            <ArrowLeft className="h-5 w-5" />
          </Button>
          <div>
            <h1 className="font-display text-lg font-bold text-foreground">
              {convo?.subject || "Podrška"}
            </h1>
            <p className="text-xs text-muted-foreground">
              {convo ? format(new Date(convo.created_at), "dd.MM.yyyy") : ""}
            </p>
          </div>
        </div>

        {/* Messages */}
        <div className="flex-1 overflow-y-auto rounded-lg border border-border bg-card p-4 space-y-3">
          {loading ? (
            <div className="flex justify-center py-16"><Loader2 className="h-8 w-8 animate-spin text-primary" /></div>
          ) : messages.length === 0 ? (
            <p className="text-center text-muted-foreground py-8">Nema poruka.</p>
          ) : (
            messages.map((msg) => {
              const isMine = msg.sender_id === user.id;
              return (
                <div key={msg.id} className={`flex ${isMine ? "justify-end" : "justify-start"}`}>
                  <div className={`max-w-[75%] rounded-2xl px-4 py-2.5 ${
                    isMine
                      ? "bg-primary text-primary-foreground rounded-br-md"
                      : "bg-muted text-foreground rounded-bl-md"
                  }`}>
                    <div className="mb-1 flex items-center gap-1.5">
                      {msg.is_admin ? (
                        <Shield className="h-3 w-3" />
                      ) : (
                        <User className="h-3 w-3" />
                      )}
                      <span className="text-[10px] font-medium opacity-70">
                        {msg.is_admin ? "BookNook Tim" : "Vi"}
                      </span>
                    </div>
                    <p className="text-sm whitespace-pre-wrap">{msg.message_text}</p>
                    <p className={`mt-1 text-[10px] text-right ${isMine ? "opacity-60" : "text-muted-foreground"}`}>
                      {format(new Date(msg.created_at), "HH:mm")}
                    </p>
                  </div>
                </div>
              );
            })
          )}
          <div ref={bottomRef} />
        </div>

        {/* Reply */}
        <div className="mt-3 flex gap-2">
          <Textarea
            value={reply}
            onChange={(e) => setReply(e.target.value)}
            onKeyDown={handleKeyDown}
            placeholder="Napišite odgovor..."
            rows={2}
            className="resize-none"
          />
          <Button onClick={handleSend} disabled={sending || !reply.trim()} className="self-end">
            <Send className="h-4 w-4" />
          </Button>
        </div>
      </div>
    </div>
  );
}
import { useState } from "react";
import { useNavigate } from "react-router-dom";
import { Upload, Loader2, FileText } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Navbar } from "@/components/Navbar";
import { FacultySelect } from "@/components/FacultySelect";
import { useAuth } from "@/lib/auth-context";
import { supabase } from "@/integrations/supabase/client";
import { toast } from "sonner";
import profileBg from "@/assets/profile-bg.jpg";

const ACCEPTED_TYPES = [
  "application/pdf",
  "application/vnd.ms-powerpoint",
  "application/vnd.openxmlformats-officedocument.presentationml.presentation",
  "application/msword",
  "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
];

const MAX_FILE_SIZE = 20 * 1024 * 1024; // 20MB

export default function UploadMaterial() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const [title, setTitle] = useState("");
  const [faculty, setFaculty] = useState("");
  const [subject, setSubject] = useState("");
  const [description, setDescription] = useState("");
  const [file, setFile] = useState<File | null>(null);
  const [uploading, setUploading] = useState(false);

  if (!user) {
    navigate("/auth");
    return null;
  }

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const selected = e.target.files?.[0];
    if (!selected) return;
    if (!ACCEPTED_TYPES.includes(selected.type)) {
      toast.error("Dozvoljeni formati: PDF, PPT, PPTX, DOC, DOCX");
      return;
    }
    if (selected.size > MAX_FILE_SIZE) {
      toast.error("Maksimalna veličina fajla je 20MB");
      return;
    }
    setFile(selected);
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!file || !title || !faculty || !subject) {
      toast.error("Popunite sva obavezna polja");
      return;
    }

    setUploading(true);
    try {
      const ext = file.name.split(".").pop();
      const path = `${user.id}/${crypto.randomUUID()}.${ext}`;

      const { error: uploadError } = await supabase.storage
        .from("study-materials")
        .upload(path, file);

      if (uploadError) throw uploadError;

      const { data: urlData } = supabase.storage
        .from("study-materials")
        .getPublicUrl(path);

      const { error: insertError } = await supabase.from("materials").insert({
        user_id: user.id,
        title,
        faculty,
        subject,
        description: description || null,
        file_url: urlData.publicUrl,
        file_name: file.name,
        file_type: file.type,
      });

      if (insertError) throw insertError;

      toast.success("Materijal je uspešno dodat!");
      navigate("/materials");
    } catch (err: any) {
      console.error(err);
      toast.error("Greška pri dodavanju materijala");
    } finally {
      setUploading(false);
    }
  };

  return (
    <div className="relative min-h-screen">
      <div className="fixed inset-0 -z-10">
        <img src={profileBg} alt="" className="h-full w-full object-cover" />
        <div className="absolute inset-0 bg-background/85 backdrop-blur-sm" />
      </div>

      <Navbar />

      <div className="container mx-auto max-w-2xl px-4 py-8">
        <Card className="border-border bg-card">
          <CardHeader>
            <CardTitle className="flex items-center gap-2 font-display text-xl">
              <Upload className="h-5 w-5 text-primary" />
              Dodaj studijski materijal
            </CardTitle>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleSubmit} className="space-y-5">
              <div className="space-y-2">
                <Label htmlFor="title">Naslov *</Label>
                <Input id="title" value={title} onChange={(e) => setTitle(e.target.value)} placeholder="Npr. Skripta iz Matematike 1" required />
              </div>

              <div className="space-y-2">
                <Label>Fakultet *</Label>
                <FacultySelect value={faculty} onValueChange={setFaculty} placeholder="Izaberite fakultet" />
              </div>

              <div className="space-y-2">
                <Label htmlFor="subject">Predmet *</Label>
                <Input id="subject" value={subject} onChange={(e) => setSubject(e.target.value)} placeholder="Npr. Matematika 1" required />
              </div>

              <div className="space-y-2">
                <Label htmlFor="description">Opis (opciono)</Label>
                <Textarea id="description" value={description} onChange={(e) => setDescription(e.target.value)} placeholder="Kratak opis materijala..." rows={3} />
              </div>

              <div className="space-y-2">
                <Label htmlFor="file">Fajl * (PDF, PPT, DOC - max 20MB)</Label>
                <div className="flex items-center gap-3">
                  <Input id="file" type="file" accept=".pdf,.ppt,.pptx,.doc,.docx" onChange={handleFileChange} className="cursor-pointer" />
                </div>
                {file && (
                  <div className="flex items-center gap-2 text-sm text-muted-foreground">
                    <FileText className="h-4 w-4" />
                    {file.name} ({(file.size / 1024 / 1024).toFixed(1)} MB)
                  </div>
                )}
              </div>

              <Button type="submit" className="w-full" disabled={uploading}>
                {uploading ? (
                  <>
                    <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                    Dodavanje...
                  </>
                ) : (
                  <>
                    <Upload className="mr-2 h-4 w-4" />
                    Dodaj materijal
                  </>
                )}
              </Button>
            </form>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
import { describe, it, expect } from "vitest";

describe("example", () => {
  it("should pass", () => {
    expect(true).toBe(true);
  });
});
import "@testing-library/jest-dom";

Object.defineProperty(window, "matchMedia", {
  writable: true,
  value: (query: string) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: () => {},
    removeListener: () => {},
    addEventListener: () => {},
    removeEventListener: () => {},
    dispatchEvent: () => {},
  }),
});
#root {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.react:hover {
  filter: drop-shadow(0 0 2em #61dafbaa);
}

@keyframes logo-spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@media (prefers-reduced-motion: no-preference) {
  a:nth-of-type(2) .logo {
    animation: logo-spin infinite 20s linear;
  }
}

.card {
  padding: 2em;
}

.read-the-docs {
  color: #888;
}
import { Toaster } from "@/components/ui/toaster";
import { Toaster as Sonner } from "@/components/ui/sonner";
import { TooltipProvider } from "@/components/ui/tooltip";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { AuthProvider } from "@/lib/auth-context";
import { ScrollToTop } from "@/components/ScrollToTop";
import Index from "./pages/Index";
import Auth from "./pages/Auth";
import Shop from "./pages/Shop";
import SellBook from "./pages/SellBook";
import Chatbot from "./pages/Chatbot";
import Profile from "./pages/Profile";
import EditBook from "./pages/EditBook";
import BookDetails from "./pages/BookDetails";
import Inbox from "./pages/Inbox";
import Conversation from "./pages/Conversation";
import Materials from "./pages/Materials";
import UploadMaterial from "./pages/UploadMaterial";
import NotFound from "./pages/NotFound";
import AdminLayout from "./pages/admin/AdminLayout";
import AdminDashboard from "./pages/admin/AdminDashboard";
import AdminListings from "./pages/admin/AdminListings";
import AdminReports from "./pages/admin/AdminReports";
import AdminUsers from "./pages/admin/AdminUsers";

import AdminSupport from "./pages/admin/AdminSupport";
import Contact from "./pages/Contact";
import Support from "./pages/Support";
import SupportConversation from "./pages/SupportConversation";

const queryClient = new QueryClient();

const App = () => (
  <QueryClientProvider client={queryClient}>
    <TooltipProvider>
      <Toaster />
      <Sonner />
      <BrowserRouter>
        <AuthProvider>
          <ScrollToTop />
          <Routes>
            <Route path="/" element={<Index />} />
            <Route path="/auth" element={<Auth />} />
            <Route path="/shop" element={<Shop />} />
            <Route path="/sell" element={<SellBook />} />
            <Route path="/chatbot" element={<Chatbot />} />
            <Route path="/profile" element={<Profile />} />
            <Route path="/edit/:id" element={<EditBook />} />
            <Route path="/listing/:id" element={<BookDetails />} />
            <Route path="/materials" element={<Materials />} />
            <Route path="/upload-material" element={<UploadMaterial />} />
            <Route path="/inbox" element={<Inbox />} />
            <Route path="/conversation/:id" element={<Conversation />} />
            <Route path="/contact" element={<Contact />} />
            <Route path="/support" element={<Support />} />
            <Route path="/support/:id" element={<SupportConversation />} />
            <Route path="/admin" element={<AdminLayout />}>
              <Route index element={<AdminDashboard />} />
              <Route path="listings" element={<AdminListings />} />
              <Route path="reports" element={<AdminReports />} />
              <Route path="users" element={<AdminUsers />} />
              
              <Route path="support" element={<AdminSupport />} />
            </Route>
            <Route path="*" element={<NotFound />} />
          </Routes>
        </AuthProvider>
      </BrowserRouter>
    </TooltipProvider>
  </QueryClientProvider>
);

export default App;
@tailwind base;
@tailwind components;
@tailwind utilities;

@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=DM+Sans:wght@400;500;600;700&display=swap');

@layer base {
  :root {
    --background: 263 80% 96%;
    --foreground: 263 40% 10%;

    --card: 263 60% 97%;
    --card-foreground: 263 40% 10%;

    --popover: 263 55% 97%;
    --popover-foreground: 263 40% 10%;

    --primary: 263 83% 57%;
    --primary-foreground: 0 0% 100%;
    --primary-dark: 256 84% 48%;

    --secondary: 263 50% 90%;
    --secondary-foreground: 263 40% 20%;

    --muted: 263 40% 91%;
    --muted-foreground: 263 15% 42%;

    --accent: 263 55% 92%;
    --accent-foreground: 263 40% 15%;

    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;

    --border: 263 50% 78%;
    --input: 263 50% 80%;
    --ring: 263 83% 57%;

    --radius: 0.75rem;

    --sidebar-background: 263 50% 93%;
    --sidebar-foreground: 263 20% 25%;
    --sidebar-primary: 263 84% 58%;
    --sidebar-primary-foreground: 0 0% 100%;
    --sidebar-accent: 263 45% 88%;
    --sidebar-accent-foreground: 263 40% 15%;
    --sidebar-border: 263 35% 85%;
    --sidebar-ring: 263 84% 58%;

    --gradient-primary: linear-gradient(135deg, hsl(263 84% 58%), hsl(280 70% 65%));
    --gradient-hero: linear-gradient(135deg, hsl(263 50% 12%), hsl(270 60% 18%), hsl(280 50% 25%));
    --gradient-card: linear-gradient(145deg, hsl(263 55% 97%), hsl(263 50% 93%));
    --shadow-primary: 0 4px 20px -4px hsl(263 84% 58% / 0.35);
    --shadow-card: 0 2px 16px -2px hsl(263 50% 50% / 0.12);
    --shadow-card-hover: 0 12px 36px -6px hsl(263 50% 50% / 0.25);
  }

  .dark {
    --background: 263 30% 6%;
    --foreground: 263 10% 95%;

    --card: 263 25% 10%;
    --card-foreground: 263 10% 95%;

    --popover: 263 25% 10%;
    --popover-foreground: 263 10% 95%;

    --primary: 263 70% 65%;
    --primary-foreground: 263 40% 8%;
    --primary-dark: 256 84% 48%;

    --secondary: 263 25% 18%;
    --secondary-foreground: 263 10% 90%;

    --muted: 263 20% 15%;
    --muted-foreground: 263 10% 60%;

    --accent: 263 30% 22%;
    --accent-foreground: 263 10% 90%;

    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;

    --border: 263 20% 18%;
    --input: 263 20% 18%;
    --ring: 263 70% 65%;
  }
}

@layer base {
  * {
    @apply border-border;
  }

  html {
    overflow-y: scroll;
    scrollbar-gutter: stable;
  }

  body {
    @apply bg-background text-foreground font-body antialiased;
    overflow-x: hidden;
  }

  h1, h2, h3, h4, h5, h6 {
    @apply font-display;
  }
}

@layer utilities {
  .gradient-primary {
    background: var(--gradient-primary);
  }
  .gradient-hero {
    background: var(--gradient-hero);
  }
  .gradient-card {
    background: var(--gradient-card);
  }
  .shadow-primary {
    box-shadow: var(--shadow-primary);
  }
  .shadow-card {
    box-shadow: var(--shadow-card);
  }
  .shadow-card-hover {
    box-shadow: var(--shadow-card-hover);
  }
  
  /* Typography utilities */
  .heading-display {
    @apply font-display text-4xl font-bold tracking-tight;
  }
  
  .heading-large {
    @apply font-display text-3xl font-bold tracking-tight;
  }
  
  .heading-medium {
    @apply font-display text-2xl font-bold tracking-tight;
  }
  
  .heading-small {
    @apply font-display text-xl font-semibold tracking-tight;
  }
  
  .nav-link {
    @apply font-body text-sm font-medium tracking-wide;
  }
  
  .button-text {
    @apply font-body text-sm font-medium;
  }
}
import { createRoot } from "react-dom/client";
import App from "./App.tsx";
import "./index.css";

createRoot(document.getElementById("root")!).render(<App />);
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type",
};

Deno.serve(async (req) => {
  if (req.method === "OPTIONS") {
    return new Response(null, { headers: corsHeaders });
  }

  try {
    const supabaseUrl = Deno.env.get("SUPABASE_URL")!;
    const serviceRoleKey = Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!;
    const supabaseAdmin = createClient(supabaseUrl, serviceRoleKey);

    // Verify the calling user is admin
    const authHeader = req.headers.get("Authorization")!;
    const token = authHeader.replace("Bearer ", "");
    const { data: { user }, error: authError } = await supabaseAdmin.auth.getUser(token);
    if (authError || !user) {
      return new Response(JSON.stringify({ error: "Unauthorized" }), {
        status: 401,
        headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    // Check admin role
    const { data: roleData } = await supabaseAdmin
      .from("user_roles")
      .select("role")
      .eq("user_id", user.id)
      .eq("role", "admin")
      .single();

    if (!roleData) {
      return new Response(JSON.stringify({ error: "Forbidden" }), {
        status: 403,
        headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    const { action, ...params } = await req.json();

    switch (action) {
      case "get_users": {
        const { data: authUsers, error } = await supabaseAdmin.auth.admin.listUsers({ perPage: 1000 });
        if (error) throw error;

        const { data: profiles } = await supabaseAdmin
          .from("profiles")
          .select("user_id, full_name, faculty, is_suspended, created_at");

        const profileMap: Record<string, any> = {};
        profiles?.forEach((p: any) => { profileMap[p.user_id] = p; });

        const users = authUsers.users.map((u: any) => ({
          id: u.id,
          email: u.email,
          created_at: u.created_at,
          full_name: profileMap[u.id]?.full_name || "",
          faculty: profileMap[u.id]?.faculty || "",
          is_suspended: profileMap[u.id]?.is_suspended || false,
        }));

        return new Response(JSON.stringify({ users }), {
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }

      case "get_stats": {
        const { count: bookCount } = await supabaseAdmin
          .from("books")
          .select("id", { count: "exact", head: true });

        const { count: reportCount } = await supabaseAdmin
          .from("reports")
          .select("id", { count: "exact", head: true })
          .eq("resolved", false);

        const { data: authUsers } = await supabaseAdmin.auth.admin.listUsers({ perPage: 1 });

        return new Response(JSON.stringify({
          stats: {
            total_listings: bookCount || 0,
            open_reports: reportCount || 0,
            total_users: authUsers?.users ? (await supabaseAdmin.auth.admin.listUsers({ perPage: 1000 })).data?.users?.length || 0 : 0,
          }
        }), {
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }

      case "suspend_user": {
        const { user_id } = params;
        await supabaseAdmin.from("profiles").update({ is_suspended: true }).eq("user_id", user_id);
        return new Response(JSON.stringify({ success: true }), {
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }

      case "reactivate_user": {
        const { user_id } = params;
        await supabaseAdmin.from("profiles").update({ is_suspended: false }).eq("user_id", user_id);
        return new Response(JSON.stringify({ success: true }), {
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }

      case "delete_book": {
        const { book_id } = params;
        // Get book to find seller_id for storage cleanup
        const { data: book } = await supabaseAdmin.from("books").select("seller_id").eq("id", book_id).single();
        if (book) {
          const { data: files } = await supabaseAdmin.storage
            .from("book-images")
            .list(`${book.seller_id}/${book_id}`);
          if (files && files.length > 0) {
            const paths = files.map((f: any) => `${book.seller_id}/${book_id}/${f.name}`);
            await supabaseAdmin.storage.from("book-images").remove(paths);
          }
        }
        await supabaseAdmin.from("books").delete().eq("id", book_id);
        return new Response(JSON.stringify({ success: true }), {
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }

      case "update_book_status": {
        const { book_id, is_sold } = params;
        await supabaseAdmin.from("books").update({ is_sold }).eq("id", book_id);
        return new Response(JSON.stringify({ success: true }), {
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }

      case "resolve_report": {
        const { report_id } = params;
        await supabaseAdmin.from("reports").update({ resolved: true }).eq("id", report_id);
        return new Response(JSON.stringify({ success: true }), {
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }

      default:
        return new Response(JSON.stringify({ error: "Unknown action" }), {
          status: 400,
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
    }
  } catch (err) {
    return new Response(JSON.stringify({ error: (err as Error).message }), {
      status: 500,
      headers: { ...corsHeaders, "Content-Type": "application/json" },
    });
  }
});
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";

const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type, x-supabase-client-platform, x-supabase-client-platform-version, x-supabase-client-runtime, x-supabase-client-runtime-version",
};

serve(async (req) => {
  if (req.method === "OPTIONS") return new Response(null, { headers: corsHeaders });

  try {
    const { messages } = await req.json();
    const LOVABLE_API_KEY = Deno.env.get("LOVABLE_API_KEY");
    if (!LOVABLE_API_KEY) throw new Error("LOVABLE_API_KEY is not configured");

    const response = await fetch("https://ai.gateway.lovable.dev/v1/chat/completions", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${LOVABLE_API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        model: "google/gemini-3-flash-preview",
        messages: [
          {
            role: "system",
            content: `Ti si BookNook AI pomoćnik - stručnjak za studentsku literaturu u Srbiji. 
Pomaži studentima da pronađu pravu literaturu za svoje fakultete i predmete.
Odgovaraj na srpskom jeziku. Budi koncizan i koristan.
Preporučuj konkretne udžbenike, autore i izdavačke kuće.
Poznajte fakultete u Srbiji (ETF, FON, Pravni, Ekonomski, Medicinski, itd).
Ako te pitaju za cenu, reci da prosečna cena polovne knjige je 500-2000 RSD.
Formatiraj odgovore koristeći markdown za bolju čitljivost.`
          },
          ...messages,
        ],
        stream: true,
      }),
    });

    if (!response.ok) {
      if (response.status === 429) {
        return new Response(JSON.stringify({ error: "Rate limit exceeded" }), {
          status: 429,
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }
      if (response.status === 402) {
        return new Response(JSON.stringify({ error: "Payment required" }), {
          status: 402,
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        });
      }
      const t = await response.text();
      console.error("AI gateway error:", response.status, t);
      return new Response(JSON.stringify({ error: "AI gateway error" }), {
        status: 500,
        headers: { ...corsHeaders, "Content-Type": "application/json" },
      });
    }

    return new Response(response.body, {
      headers: { ...corsHeaders, "Content-Type": "text/event-stream" },
    });
  } catch (e) {
    console.error("chat error:", e);
    return new Response(JSON.stringify({ error: e instanceof Error ? e.message : "Unknown error" }), {
      status: 500,
      headers: { ...corsHeaders, "Content-Type": "application/json" },
    });
  }
});

-- Create profiles table
CREATE TABLE public.profiles (
  id UUID NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID NOT NULL UNIQUE REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name TEXT,
  faculty TEXT,
  year_of_study INTEGER,
  role TEXT NOT NULL DEFAULT 'buyer' CHECK (role IN ('buyer', 'seller', 'both')),
  avatar_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now()
);

ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Profiles are viewable by everyone" ON public.profiles FOR SELECT USING (true);
CREATE POLICY "Users can insert their own profile" ON public.profiles FOR INSERT WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Users can update their own profile" ON public.profiles FOR UPDATE USING (auth.uid() = user_id);

-- Create books table
CREATE TABLE public.books (
  id UUID NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  seller_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  author TEXT NOT NULL,
  faculty TEXT NOT NULL,
  subject TEXT NOT NULL,
  year_of_study INTEGER NOT NULL CHECK (year_of_study BETWEEN 1 AND 6),
  price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
  condition TEXT NOT NULL CHECK (condition IN ('novo', 'kao novo', 'dobro', 'korisceno')),
  description TEXT,
  image_url TEXT,
  is_sold BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now()
);

ALTER TABLE public.books ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Books are viewable by everyone" ON public.books FOR SELECT USING (true);
CREATE POLICY "Sellers can insert their own books" ON public.books FOR INSERT WITH CHECK (auth.uid() = seller_id);
CREATE POLICY "Sellers can update their own books" ON public.books FOR UPDATE USING (auth.uid() = seller_id);
CREATE POLICY "Sellers can delete their own books" ON public.books FOR DELETE USING (auth.uid() = seller_id);

CREATE INDEX idx_books_faculty ON public.books(faculty);
CREATE INDEX idx_books_subject ON public.books(subject);
CREATE INDEX idx_books_year ON public.books(year_of_study);
CREATE INDEX idx_books_seller ON public.books(seller_id);

-- Create function to update timestamps
CREATE OR REPLACE FUNCTION public.update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SET search_path = public;

CREATE TRIGGER update_profiles_updated_at BEFORE UPDATE ON public.profiles FOR EACH ROW EXECUTE FUNCTION public.update_updated_at_column();
CREATE TRIGGER update_books_updated_at BEFORE UPDATE ON public.books FOR EACH ROW EXECUTE FUNCTION public.update_updated_at_column();

-- Auto-create profile on signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (user_id, full_name)
  VALUES (NEW.id, COALESCE(NEW.raw_user_meta_data->>'full_name', ''));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER SET search_path = public;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();

-- Create storage bucket for book images
INSERT INTO storage.buckets (id, name, public)
VALUES ('book-images', 'book-images', true);

-- Allow authenticated users to upload to book-images
CREATE POLICY "Authenticated users can upload book images"
ON storage.objects FOR INSERT TO authenticated
WITH CHECK (bucket_id = 'book-images');

-- Allow everyone to view book images
CREATE POLICY "Anyone can view book images"
ON storage.objects FOR SELECT
USING (bucket_id = 'book-images');

-- Allow users to delete their own uploaded images
CREATE POLICY "Users can delete their own book images"
ON storage.objects FOR DELETE TO authenticated
USING (bucket_id = 'book-images' AND (storage.foldername(name))[1] = auth.uid()::text);

CREATE TABLE public.inquiries (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  book_id uuid REFERENCES public.books(id) ON DELETE CASCADE NOT NULL,
  seller_id uuid NOT NULL,
  buyer_name text NOT NULL,
  buyer_email text NOT NULL,
  message text NOT NULL,
  created_at timestamp with time zone NOT NULL DEFAULT now()
);

ALTER TABLE public.inquiries ENABLE ROW LEVEL SECURITY;

-- Anyone can insert an inquiry (public contact form)
CREATE POLICY "Anyone can create inquiries"
ON public.inquiries
FOR INSERT
TO anon, authenticated
WITH CHECK (true);

-- Sellers can view inquiries for their listings
CREATE POLICY "Sellers can view their inquiries"
ON public.inquiries
FOR SELECT
TO authenticated
USING (auth.uid() = seller_id);

-- Create conversations table
CREATE TABLE public.conversations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  book_id uuid NOT NULL REFERENCES public.books(id) ON DELETE CASCADE,
  buyer_id uuid NOT NULL,
  seller_id uuid NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  last_message_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE (book_id, buyer_id, seller_id)
);

ALTER TABLE public.conversations ENABLE ROW LEVEL SECURITY;

-- Create messages table
CREATE TABLE public.messages (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id uuid NOT NULL REFERENCES public.conversations(id) ON DELETE CASCADE,
  sender_id uuid NOT NULL,
  message_text text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  is_read boolean NOT NULL DEFAULT false
);

ALTER TABLE public.messages ENABLE ROW LEVEL SECURITY;

-- Enable realtime for messages
ALTER PUBLICATION supabase_realtime ADD TABLE public.messages;
ALTER PUBLICATION supabase_realtime ADD TABLE public.conversations;

-- RLS: conversations visible to participants
CREATE POLICY "Participants can view conversations"
ON public.conversations FOR SELECT
TO authenticated
USING (auth.uid() = buyer_id OR auth.uid() = seller_id);

-- RLS: authenticated users can create conversations (they must be the buyer)
CREATE POLICY "Buyers can create conversations"
ON public.conversations FOR INSERT
TO authenticated
WITH CHECK (auth.uid() = buyer_id);

-- RLS: participants can update conversations (for last_message_at)
CREATE POLICY "Participants can update conversations"
ON public.conversations FOR UPDATE
TO authenticated
USING (auth.uid() = buyer_id OR auth.uid() = seller_id);

-- Helper function to check conversation participant
CREATE OR REPLACE FUNCTION public.is_conversation_participant(_user_id uuid, _conversation_id uuid)
RETURNS boolean
LANGUAGE sql
STABLE
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT EXISTS (
    SELECT 1 FROM public.conversations
    WHERE id = _conversation_id
    AND (buyer_id = _user_id OR seller_id = _user_id)
  )
$$;

-- RLS: messages visible to conversation participants
CREATE POLICY "Participants can view messages"
ON public.messages FOR SELECT
TO authenticated
USING (public.is_conversation_participant(auth.uid(), conversation_id));

-- RLS: participants can send messages
CREATE POLICY "Participants can send messages"
ON public.messages FOR INSERT
TO authenticated
WITH CHECK (
  auth.uid() = sender_id
  AND public.is_conversation_participant(auth.uid(), conversation_id)
);

-- RLS: recipients can mark messages as read
CREATE POLICY "Recipients can update message read status"
ON public.messages FOR UPDATE
TO authenticated
USING (public.is_conversation_participant(auth.uid(), conversation_id));

CREATE TABLE public.faculties (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  created_at timestamp with time zone NOT NULL DEFAULT now(),
  CONSTRAINT faculties_name_unique UNIQUE (name)
);

-- Create a unique index for case-insensitive duplicate prevention
CREATE UNIQUE INDEX faculties_name_lower_idx ON public.faculties (lower(name));

ALTER TABLE public.faculties ENABLE ROW LEVEL SECURITY;

-- Everyone can read faculties
CREATE POLICY "Faculties are viewable by everyone" ON public.faculties FOR SELECT USING (true);

-- Authenticated users can insert new faculties
CREATE POLICY "Authenticated users can add faculties" ON public.faculties FOR INSERT TO authenticated WITH CHECK (true);

-- Seed existing faculties
INSERT INTO public.faculties (name) VALUES
  ('Elektrotehnički fakultet'),
  ('Mašinski fakultet'),
  ('Pravni fakultet'),
  ('Ekonomski fakultet'),
  ('Medicinski fakultet'),
  ('Filozofski fakultet'),
  ('Matematički fakultet'),
  ('Farmaceutski fakultet'),
  ('Arhitektonski fakultet'),
  ('FON'),
  ('FPN'),
  ('Filološki fakultet');

-- Saved/bookmarked books table
CREATE TABLE public.saved_books (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL,
  book_id uuid NOT NULL REFERENCES public.books(id) ON DELETE CASCADE,
  created_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE(user_id, book_id)
);

ALTER TABLE public.saved_books ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view their own saved books"
  ON public.saved_books FOR SELECT
  TO authenticated
  USING (auth.uid() = user_id);

CREATE POLICY "Users can save books"
  ON public.saved_books FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can unsave books"
  ON public.saved_books FOR DELETE
  TO authenticated
  USING (auth.uid() = user_id);

-- Reports table
CREATE TABLE public.reports (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL,
  book_id uuid NOT NULL REFERENCES public.books(id) ON DELETE CASCADE,
  reason text NOT NULL,
  description text,
  created_at timestamptz NOT NULL DEFAULT now()
);

ALTER TABLE public.reports ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can create reports"
  ON public.reports FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can view their own reports"
  ON public.reports FOR SELECT
  TO authenticated
  USING (auth.uid() = user_id);

-- Add last_seen_at to profiles for online status
ALTER TABLE public.profiles ADD COLUMN IF NOT EXISTS last_seen_at timestamptz DEFAULT now();

-- Create app_role enum
CREATE TYPE public.app_role AS ENUM ('admin', 'moderator', 'user');

-- Create user_roles table (separate from profiles per security best practice)
CREATE TABLE public.user_roles (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  role app_role NOT NULL,
  UNIQUE (user_id, role)
);

ALTER TABLE public.user_roles ENABLE ROW LEVEL SECURITY;

-- Security definer function to check roles without recursion
CREATE OR REPLACE FUNCTION public.has_role(_user_id uuid, _role app_role)
RETURNS boolean
LANGUAGE sql
STABLE
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT EXISTS (
    SELECT 1
    FROM public.user_roles
    WHERE user_id = _user_id
      AND role = _role
  )
$$;

-- RLS: Only admins can view user_roles
CREATE POLICY "Admins can view all roles"
  ON public.user_roles FOR SELECT
  TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

-- Add resolved column to reports
ALTER TABLE public.reports ADD COLUMN IF NOT EXISTS resolved boolean NOT NULL DEFAULT false;

-- Add is_suspended column to profiles
ALTER TABLE public.profiles ADD COLUMN IF NOT EXISTS is_suspended boolean NOT NULL DEFAULT false;

-- Admin policies for books: admins can delete/update any book
CREATE POLICY "Admins can delete any book"
  ON public.books FOR DELETE
  TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

CREATE POLICY "Admins can update any book"
  ON public.books FOR UPDATE
  TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

-- Admin policies for reports: admins can view and update all reports
CREATE POLICY "Admins can view all reports"
  ON public.reports FOR SELECT
  TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

CREATE POLICY "Admins can update reports"
  ON public.reports FOR UPDATE
  TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

-- Admin policies for profiles: admins can update any profile (for suspend)
CREATE POLICY "Admins can update any profile"
  ON public.profiles FOR UPDATE
  TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

-- Create avatars storage bucket
INSERT INTO storage.buckets (id, name, public) VALUES ('avatars', 'avatars', true);

-- Allow authenticated users to upload their own avatar
CREATE POLICY "Users can upload their own avatar"
ON storage.objects FOR INSERT TO authenticated
WITH CHECK (bucket_id = 'avatars' AND (storage.foldername(name))[1] = auth.uid()::text);

-- Allow authenticated users to update their own avatar
CREATE POLICY "Users can update their own avatar"
ON storage.objects FOR UPDATE TO authenticated
USING (bucket_id = 'avatars' AND (storage.foldername(name))[1] = auth.uid()::text);

-- Allow authenticated users to delete their own avatar
CREATE POLICY "Users can delete their own avatar"
ON storage.objects FOR DELETE TO authenticated
USING (bucket_id = 'avatars' AND (storage.foldername(name))[1] = auth.uid()::text);

-- Allow public read access to avatars
CREATE POLICY "Avatars are publicly accessible"
ON storage.objects FOR SELECT
USING (bucket_id = 'avatars');

-- Create materials table
CREATE TABLE public.materials (
  id UUID NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID NOT NULL,
  title TEXT NOT NULL,
  faculty TEXT NOT NULL,
  subject TEXT NOT NULL,
  description TEXT,
  file_url TEXT NOT NULL,
  file_name TEXT NOT NULL,
  file_type TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now()
);

-- Enable RLS
ALTER TABLE public.materials ENABLE ROW LEVEL SECURITY;

-- Anyone can view materials
CREATE POLICY "Materials are viewable by everyone" ON public.materials
  FOR SELECT USING (true);

-- Authenticated users can upload
CREATE POLICY "Authenticated users can insert materials" ON public.materials
  FOR INSERT TO authenticated
  WITH CHECK (auth.uid() = user_id);

-- Users can delete their own materials
CREATE POLICY "Users can delete their own materials" ON public.materials
  FOR DELETE USING (auth.uid() = user_id);

-- Create storage bucket
INSERT INTO storage.buckets (id, name, public) VALUES ('study-materials', 'study-materials', true);

-- Storage policies
CREATE POLICY "Anyone can view study materials" ON storage.objects
  FOR SELECT USING (bucket_id = 'study-materials');

CREATE POLICY "Authenticated users can upload study materials" ON storage.objects
  FOR INSERT TO authenticated
  WITH CHECK (bucket_id = 'study-materials');

CREATE POLICY "Users can delete their own study materials" ON storage.objects
  FOR DELETE USING (bucket_id = 'study-materials' AND (storage.foldername(name))[1] = auth.uid()::text);

-- Reviews table
CREATE TABLE public.reviews (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  seller_id uuid NOT NULL,
  reviewer_id uuid NOT NULL,
  book_id uuid NOT NULL REFERENCES public.books(id) ON DELETE CASCADE,
  conversation_id uuid NOT NULL REFERENCES public.conversations(id) ON DELETE CASCADE,
  rating integer NOT NULL CHECK (rating >= 1 AND rating <= 5),
  comment text NOT NULL DEFAULT '',
  created_at timestamp with time zone NOT NULL DEFAULT now(),
  UNIQUE (reviewer_id, conversation_id)
);

ALTER TABLE public.reviews ENABLE ROW LEVEL SECURITY;

-- Security definer function to check eligibility
CREATE OR REPLACE FUNCTION public.can_review_conversation(_user_id uuid, _conversation_id uuid)
RETURNS boolean
LANGUAGE sql
STABLE
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT EXISTS (
    SELECT 1 FROM public.conversations c
    WHERE c.id = _conversation_id
    AND (c.buyer_id = _user_id OR c.seller_id = _user_id)
    AND EXISTS (
      SELECT 1 FROM public.messages m
      WHERE m.conversation_id = _conversation_id
      AND m.sender_id = _user_id
    )
  )
$$;

-- Get seller_id from conversation (to verify reviewer is NOT the seller)
CREATE OR REPLACE FUNCTION public.get_conversation_seller(_conversation_id uuid)
RETURNS uuid
LANGUAGE sql
STABLE
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT seller_id FROM public.conversations WHERE id = _conversation_id
$$;

-- RLS policies
CREATE POLICY "Reviews are viewable by everyone"
ON public.reviews FOR SELECT
USING (true);

CREATE POLICY "Eligible users can create reviews"
ON public.reviews FOR INSERT
WITH CHECK (
  auth.uid() = reviewer_id
  AND reviewer_id != seller_id
  AND can_review_conversation(auth.uid(), conversation_id)
  AND seller_id = get_conversation_seller(conversation_id)
);

CREATE TABLE public.contact_messages (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  full_name text NOT NULL,
  email text NOT NULL,
  subject text NOT NULL,
  message text NOT NULL,
  created_at timestamp with time zone NOT NULL DEFAULT now()
);

ALTER TABLE public.contact_messages ENABLE ROW LEVEL SECURITY;

-- Anyone can insert
CREATE POLICY "Anyone can submit contact messages"
  ON public.contact_messages FOR INSERT
  WITH CHECK (true);

-- Only admins can read
CREATE POLICY "Admins can view contact messages"
  ON public.contact_messages FOR SELECT
  USING (public.has_role(auth.uid(), 'admin'::app_role));

-- Allow users to see their own roles (fixes chicken-and-egg problem)
CREATE POLICY "Users can view their own roles"
ON public.user_roles FOR SELECT
TO authenticated
USING (auth.uid() = user_id);

-- Insert admin roles for the three admin emails
INSERT INTO public.user_roles (user_id, role)
SELECT id, 'admin'::app_role
FROM auth.users
WHERE email IN (
  'iva.lalevic006@gmail.com',
  'nikolinalesevic006@gmail.com',
  'miletictara77@gmail.com'
)
ON CONFLICT (user_id, role) DO NOTHING;

-- Support conversations table
CREATE TABLE public.support_conversations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL,
  subject text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  last_message_at timestamptz NOT NULL DEFAULT now()
);

ALTER TABLE public.support_conversations ENABLE ROW LEVEL SECURITY;

-- Users can view their own support conversations
CREATE POLICY "Users can view own support conversations"
  ON public.support_conversations FOR SELECT TO authenticated
  USING (auth.uid() = user_id);

-- Users can create support conversations
CREATE POLICY "Users can create support conversations"
  ON public.support_conversations FOR INSERT TO authenticated
  WITH CHECK (auth.uid() = user_id);

-- Users can update their own support conversations (for last_message_at)
CREATE POLICY "Users can update own support conversations"
  ON public.support_conversations FOR UPDATE TO authenticated
  USING (auth.uid() = user_id);

-- Admins can view all support conversations
CREATE POLICY "Admins can view all support conversations"
  ON public.support_conversations FOR SELECT TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

-- Admins can update any support conversation
CREATE POLICY "Admins can update any support conversation"
  ON public.support_conversations FOR UPDATE TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

-- Support messages table
CREATE TABLE public.support_messages (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id uuid NOT NULL REFERENCES public.support_conversations(id) ON DELETE CASCADE,
  sender_id uuid NOT NULL,
  message_text text NOT NULL,
  is_admin boolean NOT NULL DEFAULT false,
  is_read boolean NOT NULL DEFAULT false,
  created_at timestamptz NOT NULL DEFAULT now()
);

ALTER TABLE public.support_messages ENABLE ROW LEVEL SECURITY;

-- Function to check if user is participant of support conversation
CREATE OR REPLACE FUNCTION public.is_support_participant(_user_id uuid, _conversation_id uuid)
RETURNS boolean
LANGUAGE sql
STABLE
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT EXISTS (
    SELECT 1 FROM public.support_conversations
    WHERE id = _conversation_id AND user_id = _user_id
  )
$$;

-- Users can view messages in their own conversations
CREATE POLICY "Users can view own support messages"
  ON public.support_messages FOR SELECT TO authenticated
  USING (public.is_support_participant(auth.uid(), conversation_id));

-- Users can send messages in their own conversations
CREATE POLICY "Users can send support messages"
  ON public.support_messages FOR INSERT TO authenticated
  WITH CHECK (auth.uid() = sender_id AND public.is_support_participant(auth.uid(), conversation_id));

-- Users can update read status
CREATE POLICY "Users can update support message read status"
  ON public.support_messages FOR UPDATE TO authenticated
  USING (public.is_support_participant(auth.uid(), conversation_id));

-- Admins can view all support messages
CREATE POLICY "Admins can view all support messages"
  ON public.support_messages FOR SELECT TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

-- Admins can send messages to any conversation
CREATE POLICY "Admins can send support messages"
  ON public.support_messages FOR INSERT TO authenticated
  WITH CHECK (auth.uid() = sender_id AND public.has_role(auth.uid(), 'admin'));

-- Admins can update any support message
CREATE POLICY "Admins can update support messages"
  ON public.support_messages FOR UPDATE TO authenticated
  USING (public.has_role(auth.uid(), 'admin'));

-- Enable realtime for support messages
ALTER PUBLICATION supabase_realtime ADD TABLE public.support_messages;
project_id = "odlrghupxkfdhvclevae"

[functions.book-chat]
verify_jwt = false
VITE_SUPABASE_PROJECT_ID="odlrghupxkfdhvclevae"
VITE_SUPABASE_PUBLISHABLE_KEY="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im9kbHJnaHVweGtmZGh2Y2xldmFlIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzI3MjU1NjgsImV4cCI6MjA4ODMwMTU2OH0.FKqwQAskWW7JMqm6fmhxhjSnMTm-hTlRNwhajk20uoI"
VITE_SUPABASE_URL="https://odlrghupxkfdhvclevae.supabase.co"
# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

node_modules
dist
dist-ssr
*.local

# Editor directories and files
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
{
  "lockfileVersion": 1,
  "configVersion": 1,
  "workspaces": {
    "": {
      "name": "vite_react_shadcn_ts",
      "dependencies": {
        "@hookform/resolvers": "^3.10.0",
        "@radix-ui/react-accordion": "^1.2.11",
        "@radix-ui/react-alert-dialog": "^1.1.14",
        "@radix-ui/react-aspect-ratio": "^1.1.7",
        "@radix-ui/react-avatar": "^1.1.10",
        "@radix-ui/react-checkbox": "^1.3.2",
        "@radix-ui/react-collapsible": "^1.1.11",
        "@radix-ui/react-context-menu": "^2.2.15",
        "@radix-ui/react-dialog": "^1.1.14",
        "@radix-ui/react-dropdown-menu": "^2.1.15",
        "@radix-ui/react-hover-card": "^1.1.14",
        "@radix-ui/react-label": "^2.1.7",
        "@radix-ui/react-menubar": "^1.1.15",
        "@radix-ui/react-navigation-menu": "^1.2.13",
        "@radix-ui/react-popover": "^1.1.14",
        "@radix-ui/react-progress": "^1.1.7",
        "@radix-ui/react-radio-group": "^1.3.7",
        "@radix-ui/react-scroll-area": "^1.2.9",
        "@radix-ui/react-select": "^2.2.5",
        "@radix-ui/react-separator": "^1.1.7",
        "@radix-ui/react-slider": "^1.3.5",
        "@radix-ui/react-slot": "^1.2.3",
        "@radix-ui/react-switch": "^1.2.5",
        "@radix-ui/react-tabs": "^1.1.12",
        "@radix-ui/react-toast": "^1.2.14",
        "@radix-ui/react-toggle": "^1.1.9",
        "@radix-ui/react-toggle-group": "^1.1.10",
        "@radix-ui/react-tooltip": "^1.2.7",
        "@supabase/supabase-js": "^2.98.0",
        "@tanstack/react-query": "^5.83.0",
        "class-variance-authority": "^0.7.1",
        "clsx": "^2.1.1",
        "cmdk": "^1.1.1",
        "date-fns": "^3.6.0",
        "embla-carousel-react": "^8.6.0",
        "framer-motion": "^12.35.0",
        "input-otp": "^1.4.2",
        "lucide-react": "^0.462.0",
        "next-themes": "^0.3.0",
        "react": "^18.3.1",
        "react-day-picker": "^8.10.1",
        "react-dom": "^18.3.1",
        "react-hook-form": "^7.61.1",
        "react-markdown": "^10.1.0",
        "react-resizable-panels": "^2.1.9",
        "react-router-dom": "^6.30.1",
        "recharts": "^2.15.4",
        "sonner": "^1.7.4",
        "tailwind-merge": "^2.6.0",
        "tailwindcss-animate": "^1.0.7",
        "vaul": "^0.9.9",
        "zod": "^3.25.76",
      },
      "devDependencies": {
        "@eslint/js": "^9.32.0",
        "@tailwindcss/typography": "^0.5.16",
        "@testing-library/jest-dom": "^6.6.0",
        "@testing-library/react": "^16.0.0",
        "@types/node": "^22.16.5",
        "@types/react": "^18.3.23",
        "@types/react-dom": "^18.3.7",
        "@vitejs/plugin-react-swc": "^3.11.0",
        "autoprefixer": "^10.4.21",
        "eslint": "^9.32.0",
        "eslint-plugin-react-hooks": "^5.2.0",
        "eslint-plugin-react-refresh": "^0.4.20",
        "globals": "^15.15.0",
        "jsdom": "^20.0.3",
        "lovable-tagger": "^1.1.13",
        "postcss": "^8.5.6",
        "tailwindcss": "^3.4.17",
        "typescript": "^5.8.3",
        "typescript-eslint": "^8.38.0",
        "vite": "^5.4.19",
        "vitest": "^3.2.4",
      },
    },
  },
  "packages": {
    "@adobe/css-tools": ["@adobe/css-tools@4.4.4", "", {}, "sha512-Elp+iwUx5rN5+Y8xLt5/GRoG20WGoDCQ/1Fb+1LiGtvwbDavuSk0jhD/eZdckHAuzcDzccnkv+rEjyWfRx18gg=="],

    "@alloc/quick-lru": ["@alloc/quick-lru@5.2.0", "", {}, "sha512-UrcABB+4bUrFABwbluTIBErXwvbsU/V7TZWfmbgJfbkwiBuziS9gxdODUyuiecfdGQ85jglMW6juS3+z5TsKLw=="],

    "@babel/runtime": ["@babel/runtime@7.28.2", "", {}, "sha512-KHp2IflsnGywDjBWDkR9iEqiWSpc8GIi0lgTT3mOElT0PP1tG26P4tmFI2YvAdzgq9RGyoHZQEIEdZy6Ec5xCA=="],

    "@esbuild/aix-ppc64": ["@esbuild/aix-ppc64@0.25.0", "", { "os": "aix", "cpu": "ppc64" }, "sha512-O7vun9Sf8DFjH2UtqK8Ku3LkquL9SZL8OLY1T5NZkA34+wG3OQF7cl4Ql8vdNzM6fzBbYfLaiRLIOZ+2FOCgBQ=="],

    "@esbuild/android-arm": ["@esbuild/android-arm@0.25.0", "", { "os": "android", "cpu": "arm" }, "sha512-PTyWCYYiU0+1eJKmw21lWtC+d08JDZPQ5g+kFyxP0V+es6VPPSUhM6zk8iImp2jbV6GwjX4pap0JFbUQN65X1g=="],

    "@esbuild/android-arm64": ["@esbuild/android-arm64@0.25.0", "", { "os": "android", "cpu": "arm64" }, "sha512-grvv8WncGjDSyUBjN9yHXNt+cq0snxXbDxy5pJtzMKGmmpPxeAmAhWxXI+01lU5rwZomDgD3kJwulEnhTRUd6g=="],

    "@esbuild/android-x64": ["@esbuild/android-x64@0.25.0", "", { "os": "android", "cpu": "x64" }, "sha512-m/ix7SfKG5buCnxasr52+LI78SQ+wgdENi9CqyCXwjVR2X4Jkz+BpC3le3AoBPYTC9NHklwngVXvbJ9/Akhrfg=="],

    "@esbuild/darwin-arm64": ["@esbuild/darwin-arm64@0.25.0", "", { "os": "darwin", "cpu": "arm64" }, "sha512-mVwdUb5SRkPayVadIOI78K7aAnPamoeFR2bT5nszFUZ9P8UpK4ratOdYbZZXYSqPKMHfS1wdHCJk1P1EZpRdvw=="],

    "@esbuild/darwin-x64": ["@esbuild/darwin-x64@0.25.0", "", { "os": "darwin", "cpu": "x64" }, "sha512-DgDaYsPWFTS4S3nWpFcMn/33ZZwAAeAFKNHNa1QN0rI4pUjgqf0f7ONmXf6d22tqTY+H9FNdgeaAa+YIFUn2Rg=="],

    "@esbuild/freebsd-arm64": ["@esbuild/freebsd-arm64@0.25.0", "", { "os": "freebsd", "cpu": "arm64" }, "sha512-VN4ocxy6dxefN1MepBx/iD1dH5K8qNtNe227I0mnTRjry8tj5MRk4zprLEdG8WPyAPb93/e4pSgi1SoHdgOa4w=="],

    "@esbuild/freebsd-x64": ["@esbuild/freebsd-x64@0.25.0", "", { "os": "freebsd", "cpu": "x64" }, "sha512-mrSgt7lCh07FY+hDD1TxiTyIHyttn6vnjesnPoVDNmDfOmggTLXRv8Id5fNZey1gl/V2dyVK1VXXqVsQIiAk+A=="],

    "@esbuild/linux-arm": ["@esbuild/linux-arm@0.25.0", "", { "os": "linux", "cpu": "arm" }, "sha512-vkB3IYj2IDo3g9xX7HqhPYxVkNQe8qTK55fraQyTzTX/fxaDtXiEnavv9geOsonh2Fd2RMB+i5cbhu2zMNWJwg=="],

    "@esbuild/linux-arm64": ["@esbuild/linux-arm64@0.25.0", "", { "os": "linux", "cpu": "arm64" }, "sha512-9QAQjTWNDM/Vk2bgBl17yWuZxZNQIF0OUUuPZRKoDtqF2k4EtYbpyiG5/Dk7nqeK6kIJWPYldkOcBqjXjrUlmg=="],

    "@esbuild/linux-ia32": ["@esbuild/linux-ia32@0.25.0", "", { "os": "linux", "cpu": "ia32" }, "sha512-43ET5bHbphBegyeqLb7I1eYn2P/JYGNmzzdidq/w0T8E2SsYL1U6un2NFROFRg1JZLTzdCoRomg8Rvf9M6W6Gg=="],

    "@esbuild/linux-loong64": ["@esbuild/linux-loong64@0.25.0", "", { "os": "linux", "cpu": "none" }, "sha512-fC95c/xyNFueMhClxJmeRIj2yrSMdDfmqJnyOY4ZqsALkDrrKJfIg5NTMSzVBr5YW1jf+l7/cndBfP3MSDpoHw=="],

    "@esbuild/linux-mips64el": ["@esbuild/linux-mips64el@0.25.0", "", { "os": "linux", "cpu": "none" }, "sha512-nkAMFju7KDW73T1DdH7glcyIptm95a7Le8irTQNO/qtkoyypZAnjchQgooFUDQhNAy4iu08N79W4T4pMBwhPwQ=="],

    "@esbuild/linux-ppc64": ["@esbuild/linux-ppc64@0.25.0", "", { "os": "linux", "cpu": "ppc64" }, "sha512-NhyOejdhRGS8Iwv+KKR2zTq2PpysF9XqY+Zk77vQHqNbo/PwZCzB5/h7VGuREZm1fixhs4Q/qWRSi5zmAiO4Fw=="],

    "@esbuild/linux-riscv64": ["@esbuild/linux-riscv64@0.25.0", "", { "os": "linux", "cpu": "none" }, "sha512-5S/rbP5OY+GHLC5qXp1y/Mx//e92L1YDqkiBbO9TQOvuFXM+iDqUNG5XopAnXoRH3FjIUDkeGcY1cgNvnXp/kA=="],

    "@esbuild/linux-s390x": ["@esbuild/linux-s390x@0.25.0", "", { "os": "linux", "cpu": "s390x" }, "sha512-XM2BFsEBz0Fw37V0zU4CXfcfuACMrppsMFKdYY2WuTS3yi8O1nFOhil/xhKTmE1nPmVyvQJjJivgDT+xh8pXJA=="],

    "@esbuild/linux-x64": ["@esbuild/linux-x64@0.25.0", "", { "os": "linux", "cpu": "x64" }, "sha512-9yl91rHw/cpwMCNytUDxwj2XjFpxML0y9HAOH9pNVQDpQrBxHy01Dx+vaMu0N1CKa/RzBD2hB4u//nfc+Sd3Cw=="],

    "@esbuild/netbsd-arm64": ["@esbuild/netbsd-arm64@0.25.0", "", { "os": "none", "cpu": "arm64" }, "sha512-RuG4PSMPFfrkH6UwCAqBzauBWTygTvb1nxWasEJooGSJ/NwRw7b2HOwyRTQIU97Hq37l3npXoZGYMy3b3xYvPw=="],

    "@esbuild/netbsd-x64": ["@esbuild/netbsd-x64@0.25.0", "", { "os": "none", "cpu": "x64" }, "sha512-jl+qisSB5jk01N5f7sPCsBENCOlPiS/xptD5yxOx2oqQfyourJwIKLRA2yqWdifj3owQZCL2sn6o08dBzZGQzA=="],

    "@esbuild/openbsd-arm64": ["@esbuild/openbsd-arm64@0.25.0", "", { "os": "openbsd", "cpu": "arm64" }, "sha512-21sUNbq2r84YE+SJDfaQRvdgznTD8Xc0oc3p3iW/a1EVWeNj/SdUCbm5U0itZPQYRuRTW20fPMWMpcrciH2EJw=="],

    "@esbuild/openbsd-x64": ["@esbuild/openbsd-x64@0.25.0", "", { "os": "openbsd", "cpu": "x64" }, "sha512-2gwwriSMPcCFRlPlKx3zLQhfN/2WjJ2NSlg5TKLQOJdV0mSxIcYNTMhk3H3ulL/cak+Xj0lY1Ym9ysDV1igceg=="],

    "@esbuild/sunos-x64": ["@esbuild/sunos-x64@0.25.0", "", { "os": "sunos", "cpu": "x64" }, "sha512-bxI7ThgLzPrPz484/S9jLlvUAHYMzy6I0XiU1ZMeAEOBcS0VePBFxh1JjTQt3Xiat5b6Oh4x7UC7IwKQKIJRIg=="],

    "@esbuild/win32-arm64": ["@esbuild/win32-arm64@0.25.0", "", { "os": "win32", "cpu": "arm64" }, "sha512-ZUAc2YK6JW89xTbXvftxdnYy3m4iHIkDtK3CLce8wg8M2L+YZhIvO1DKpxrd0Yr59AeNNkTiic9YLf6FTtXWMw=="],

    "@esbuild/win32-ia32": ["@esbuild/win32-ia32@0.25.0", "", { "os": "win32", "cpu": "ia32" }, "sha512-eSNxISBu8XweVEWG31/JzjkIGbGIJN/TrRoiSVZwZ6pkC6VX4Im/WV2cz559/TXLcYbcrDN8JtKgd9DJVIo8GA=="],

    "@esbuild/win32-x64": ["@esbuild/win32-x64@0.25.0", "", { "os": "win32", "cpu": "x64" }, "sha512-ZENoHJBxA20C2zFzh6AI4fT6RraMzjYw4xKWemRTRmRVtN9c5DcH9r/f2ihEkMjOW5eGgrwCslG/+Y/3bL+DHQ=="],

    "@eslint-community/eslint-utils": ["@eslint-community/eslint-utils@4.7.0", "", { "dependencies": { "eslint-visitor-keys": "^3.4.3" }, "peerDependencies": { "eslint": "^6.0.0 || ^7.0.0 || >=8.0.0" } }, "sha512-dyybb3AcajC7uha6CvhdVRJqaKyn7w2YKqKyAN37NKYgZT36w+iRb0Dymmc5qEJ549c/S31cMMSFd75bteCpCw=="],

    "@eslint-community/regexpp": ["@eslint-community/regexpp@4.12.1", "", {}, "sha512-CCZCDJuduB9OUkFkY2IgppNZMi2lBQgD2qzwXkEia16cge2pijY/aXi96CJMquDMn3nJdlPV1A5KrJEXwfLNzQ=="],

    "@eslint/config-array": ["@eslint/config-array@0.21.0", "", { "dependencies": { "@eslint/object-schema": "^2.1.6", "debug": "^4.3.1", "minimatch": "^3.1.2" } }, "sha512-ENIdc4iLu0d93HeYirvKmrzshzofPw6VkZRKQGe9Nv46ZnWUzcF1xV01dcvEg/1wXUR61OmmlSfyeyO7EvjLxQ=="],

    "@eslint/config-helpers": ["@eslint/config-helpers@0.3.0", "", {}, "sha512-ViuymvFmcJi04qdZeDc2whTHryouGcDlaxPqarTD0ZE10ISpxGUVZGZDx4w01upyIynL3iu6IXH2bS1NhclQMw=="],

    "@eslint/core": ["@eslint/core@0.15.1", "", { "dependencies": { "@types/json-schema": "^7.0.15" } }, "sha512-bkOp+iumZCCbt1K1CmWf0R9pM5yKpDv+ZXtvSyQpudrI9kuFLp+bM2WOPXImuD/ceQuaa8f5pj93Y7zyECIGNA=="],

    "@eslint/eslintrc": ["@eslint/eslintrc@3.3.1", "", { "dependencies": { "ajv": "^6.12.4", "debug": "^4.3.2", "espree": "^10.0.1", "globals": "^14.0.0", "ignore": "^5.2.0", "import-fresh": "^3.2.1", "js-yaml": "^4.1.0", "minimatch": "^3.1.2", "strip-json-comments": "^3.1.1" } }, "sha512-gtF186CXhIl1p4pJNGZw8Yc6RlshoePRvE0X91oPGb3vZ8pM3qOS9W9NGPat9LziaBV7XrJWGylNQXkGcnM3IQ=="],

    "@eslint/js": ["@eslint/js@9.32.0", "", {}, "sha512-BBpRFZK3eX6uMLKz8WxFOBIFFcGFJ/g8XuwjTHCqHROSIsopI+ddn/d5Cfh36+7+e5edVS8dbSHnBNhrLEX0zg=="],

    "@eslint/object-schema": ["@eslint/object-schema@2.1.6", "", {}, "sha512-RBMg5FRL0I0gs51M/guSAj5/e14VQ4tpZnQNWwuDT66P14I43ItmPfIZRhO9fUVIPOAQXU47atlywZ/czoqFPA=="],

    "@eslint/plugin-kit": ["@eslint/plugin-kit@0.3.4", "", { "dependencies": { "@eslint/core": "^0.15.1", "levn": "^0.4.1" } }, "sha512-Ul5l+lHEcw3L5+k8POx6r74mxEYKG5kOb6Xpy2gCRW6zweT6TEhAf8vhxGgjhqrd/VO/Dirhsb+1hNpD1ue9hw=="],

    "@floating-ui/core": ["@floating-ui/core@1.7.2", "", { "dependencies": { "@floating-ui/utils": "^0.2.10" } }, "sha512-wNB5ooIKHQc+Kui96jE/n69rHFWAVoxn5CAzL1Xdd8FG03cgY3MLO+GF9U3W737fYDSgPWA6MReKhBQBop6Pcw=="],

    "@floating-ui/dom": ["@floating-ui/dom@1.7.2", "", { "dependencies": { "@floating-ui/core": "^1.7.2", "@floating-ui/utils": "^0.2.10" } }, "sha512-7cfaOQuCS27HD7DX+6ib2OrnW+b4ZBwDNnCcT0uTyidcmyWb03FnQqJybDBoCnpdxwBSfA94UAYlRCt7mV+TbA=="],

    "@floating-ui/react-dom": ["@floating-ui/react-dom@2.1.4", "", { "dependencies": { "@floating-ui/dom": "^1.7.2" }, "peerDependencies": { "react": ">=16.8.0", "react-dom": ">=16.8.0" } }, "sha512-JbbpPhp38UmXDDAu60RJmbeme37Jbgsm7NrHGgzYYFKmblzRUh6Pa641dII6LsjwF4XlScDrde2UAzDo/b9KPw=="],

    "@floating-ui/utils": ["@floating-ui/utils@0.2.10", "", {}, "sha512-aGTxbpbg8/b5JfU1HXSrbH3wXZuLPJcNEcZQFMxLs3oSzgtVu6nFPkbbGGUvBcUjKV2YyB9Wxxabo+HEH9tcRQ=="],

    "@hookform/resolvers": ["@hookform/resolvers@3.10.0", "", { "peerDependencies": { "react-hook-form": "^7.0.0" } }, "sha512-79Dv+3mDF7i+2ajj7SkypSKHhl1cbln1OGavqrsF7p6mbUv11xpqpacPsGDCTRvCSjEEIez2ef1NveSVL3b0Ag=="],

    "@humanfs/core": ["@humanfs/core@0.19.1", "", {}, "sha512-5DyQ4+1JEUzejeK1JGICcideyfUbGixgS9jNgex5nqkW+cY7WZhxBigmieN5Qnw9ZosSNVC9KQKyb+GUaGyKUA=="],

    "@humanfs/node": ["@humanfs/node@0.16.6", "", { "dependencies": { "@humanfs/core": "^0.19.1", "@humanwhocodes/retry": "^0.3.0" } }, "sha512-YuI2ZHQL78Q5HbhDiBA1X4LmYdXCKCMQIfw0pw7piHJwyREFebJUvrQN4cMssyES6x+vfUbx1CIpaQUKYdQZOw=="],

    "@humanwhocodes/module-importer": ["@humanwhocodes/module-importer@1.0.1", "", {}, "sha512-bxveV4V8v5Yb4ncFTT3rPSgZBOpCkjfK0y4oVVVJwIuDVBRMDXrPyXRL988i5ap9m9bnyEEjWfm5WkBmtffLfA=="],

    "@humanwhocodes/retry": ["@humanwhocodes/retry@0.4.3", "", {}, "sha512-bV0Tgo9K4hfPCek+aMAn81RppFKv2ySDQeMoSZuvTASywNTnVJCArCZE2FWqpvIatKu7VMRLWlR1EazvVhDyhQ=="],

    "@isaacs/cliui": ["@isaacs/cliui@8.0.2", "", { "dependencies": { "string-width": "^5.1.2", "string-width-cjs": "npm:string-width@^4.2.0", "strip-ansi": "^7.0.1", "strip-ansi-cjs": "npm:strip-ansi@^6.0.1", "wrap-ansi": "^8.1.0", "wrap-ansi-cjs": "npm:wrap-ansi@^7.0.0" } }, "sha512-O8jcjabXaleOG9DQ0+ARXWZBTfnP4WNAqzuiJK7ll44AmxGKv/J2M4TPjxjY3znBCfvBXFzucm1twdyFybFqEA=="],

    "@jridgewell/gen-mapping": ["@jridgewell/gen-mapping@0.3.5", "", { "dependencies": { "@jridgewell/set-array": "^1.2.1", "@jridgewell/sourcemap-codec": "^1.4.10", "@jridgewell/trace-mapping": "^0.3.24" } }, "sha512-IzL8ZoEDIBRWEzlCcRhOaCupYyN5gdIK+Q6fbFdPDg6HqX6jpkItn7DFIpW9LQzXG6Df9sA7+OKnq0qlz/GaQg=="],

    "@jridgewell/resolve-uri": ["@jridgewell/resolve-uri@3.1.2", "", {}, "sha512-bRISgCIjP20/tbWSPWMEi54QVPRZExkuD9lJL+UIxUKtwVJA8wW1Trb1jMs1RFXo1CBTNZ/5hpC9QvmKWdopKw=="],

    "@jridgewell/set-array": ["@jridgewell/set-array@1.2.1", "", {}, "sha512-R8gLRTZeyp03ymzP/6Lil/28tGeGEzhx1q2k703KGWRAI1VdvPIXdG70VJc2pAMw3NA6JKL5hhFu1sJX0Mnn/A=="],

    "@jridgewell/sourcemap-codec": ["@jridgewell/sourcemap-codec@1.5.5", "", {}, "sha512-cYQ9310grqxueWbl+WuIUIaiUaDcj7WOq5fVhEljNVgRfOUhY9fy2zTvfoqWsnebh8Sl70VScFbICvJnLKB0Og=="],

    "@jridgewell/trace-mapping": ["@jridgewell/trace-mapping@0.3.25", "", { "dependencies": { "@jridgewell/resolve-uri": "^3.1.0", "@jridgewell/sourcemap-codec": "^1.4.14" } }, "sha512-vNk6aEwybGtawWmy/PzwnGDOjCkLWSD2wqvjGGAgOAwCGWySYXfYoxt00IJkTF+8Lb57DwOb3Aa0o9CApepiYQ=="],

    "@nodelib/fs.scandir": ["@nodelib/fs.scandir@2.1.5", "", { "dependencies": { "@nodelib/fs.stat": "2.0.5", "run-parallel": "^1.1.9" } }, "sha512-vq24Bq3ym5HEQm2NKCr3yXDwjc7vTsEThRDnkp2DK9p1uqLR+DHurm/NOTo0KG7HYHU7eppKZj3MyqYuMBf62g=="],

    "@nodelib/fs.stat": ["@nodelib/fs.stat@2.0.5", "", {}, "sha512-RkhPPp2zrqDAQA/2jNhnztcPAlv64XdhIp7a7454A5ovI7Bukxgt7MX7udwAu3zg1DcpPU0rz3VV1SeaqvY4+A=="],

    "@nodelib/fs.walk": ["@nodelib/fs.walk@1.2.8", "", { "dependencies": { "@nodelib/fs.scandir": "2.1.5", "fastq": "^1.6.0" } }, "sha512-oGB+UxlgWcgQkgwo8GcEGwemoTFt3FIO9ababBmaGwXIoBKZ+GTy0pP185beGg7Llih/NSHSV2XAs1lnznocSg=="],

    "@pkgjs/parseargs": ["@pkgjs/parseargs@0.11.0", "", {}, "sha512-+1VkjdD0QBLPodGrJUeqarH8VAIvQODIbwh9XpP5Syisf7YoQgsJKPNFoqqLQlu+VQ/tVSshMR6loPMn8U+dPg=="],

    "@radix-ui/number": ["@radix-ui/number@1.1.1", "", {}, "sha512-MkKCwxlXTgz6CFoJx3pCwn07GKp36+aZyu/u2Ln2VrA5DcdyCZkASEDBTd8x5whTQQL5CiYf4prXKLcgQdv29g=="],

    "@radix-ui/primitive": ["@radix-ui/primitive@1.1.2", "", {}, "sha512-XnbHrrprsNqZKQhStrSwgRUQzoCI1glLzdw79xiZPoofhGICeZRSQ3dIxAKH1gb3OHfNf4d6f+vAv3kil2eggA=="],

    "@radix-ui/react-accordion": ["@radix-ui/react-accordion@1.2.11", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-collapsible": "1.1.11", "@radix-ui/react-collection": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-l3W5D54emV2ues7jjeG1xcyN7S3jnK3zE2zHqgn0CmMsy9lNJwmgcrmaxS+7ipw15FAivzKNzH3d5EcGoFKw0A=="],

    "@radix-ui/react-alert-dialog": ["@radix-ui/react-alert-dialog@1.1.14", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-dialog": "1.1.14", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-slot": "1.2.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-IOZfZ3nPvN6lXpJTBCunFQPRSvK8MDgSc1FB85xnIpUKOw9en0dJj8JmCAxV7BiZdtYlUpmrQjoTFkVYtdoWzQ=="],

    "@radix-ui/react-arrow": ["@radix-ui/react-arrow@1.1.7", "", { "dependencies": { "@radix-ui/react-primitive": "2.1.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-F+M1tLhO+mlQaOWspE8Wstg+z6PwxwRd8oQ8IXceWz92kfAmalTRf0EjrouQeo7QssEPfCn05B4Ihs1K9WQ/7w=="],

    "@radix-ui/react-aspect-ratio": ["@radix-ui/react-aspect-ratio@1.1.7", "", { "dependencies": { "@radix-ui/react-primitive": "2.1.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-Yq6lvO9HQyPwev1onK1daHCHqXVLzPhSVjmsNjCa2Zcxy2f7uJD2itDtxknv6FzAKCwD1qQkeVDmX/cev13n/g=="],

    "@radix-ui/react-avatar": ["@radix-ui/react-avatar@1.1.10", "", { "dependencies": { "@radix-ui/react-context": "1.1.2", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-is-hydrated": "0.1.0", "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-V8piFfWapM5OmNCXTzVQY+E1rDa53zY+MQ4Y7356v4fFz6vqCyUtIz2rUD44ZEdwg78/jKmMJHj07+C/Z/rcog=="],

    "@radix-ui/react-checkbox": ["@radix-ui/react-checkbox@1.3.2", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-use-previous": "1.1.1", "@radix-ui/react-use-size": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-yd+dI56KZqawxKZrJ31eENUwqc1QSqg4OZ15rybGjF2ZNwMO+wCyHzAVLRp9qoYJf7kYy0YpZ2b0JCzJ42HZpA=="],

    "@radix-ui/react-collapsible": ["@radix-ui/react-collapsible@1.1.11", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-2qrRsVGSCYasSz1RFOorXwl0H7g7J1frQtgpQgYrt+MOidtPAINHn9CPovQXb83r8ahapdx3Tu0fa/pdFFSdPg=="],

    "@radix-ui/react-collection": ["@radix-ui/react-collection@1.1.7", "", { "dependencies": { "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-slot": "1.2.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-Fh9rGN0MoI4ZFUNyfFVNU4y9LUz93u9/0K+yLgA2bwRojxM8JU1DyvvMBabnZPBgMWREAJvU2jjVzq+LrFUglw=="],

    "@radix-ui/react-compose-refs": ["@radix-ui/react-compose-refs@1.1.2", "", { "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-z4eqJvfiNnFMHIIvXP3CY57y2WJs5g2v3X0zm9mEJkrkNv4rDxu+sg9Jh8EkXyeqBkB7SOcboo9dMVqhyrACIg=="],

    "@radix-ui/react-context": ["@radix-ui/react-context@1.1.2", "", { "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-jCi/QKUM2r1Ju5a3J64TH2A5SpKAgh0LpknyqdQ4m6DCV0xJ2HG1xARRwNGPQfi1SLdLWZ1OJz6F4OMBBNiGJA=="],

    "@radix-ui/react-context-menu": ["@radix-ui/react-context-menu@2.2.15", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-menu": "2.1.15", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-UsQUMjcYTsBjTSXw0P3GO0werEQvUY2plgRQuKoCTtkNr45q1DiL51j4m7gxhABzZ0BadoXNsIbg7F3KwiUBbw=="],

    "@radix-ui/react-dialog": ["@radix-ui/react-dialog@1.1.14", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-dismissable-layer": "1.1.10", "@radix-ui/react-focus-guards": "1.1.2", "@radix-ui/react-focus-scope": "1.1.7", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-portal": "1.1.9", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-slot": "1.2.3", "@radix-ui/react-use-controllable-state": "1.2.2", "aria-hidden": "^1.2.4", "react-remove-scroll": "^2.6.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-+CpweKjqpzTmwRwcYECQcNYbI8V9VSQt0SNFKeEBLgfucbsLssU6Ppq7wUdNXEGb573bMjFhVjKVll8rmV6zMw=="],

    "@radix-ui/react-direction": ["@radix-ui/react-direction@1.1.1", "", { "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-1UEWRX6jnOA2y4H5WczZ44gOOjTEmlqv1uNW4GAJEO5+bauCBhv8snY65Iw5/VOS/ghKN9gr2KjnLKxrsvoMVw=="],

    "@radix-ui/react-dismissable-layer": ["@radix-ui/react-dismissable-layer@1.1.10", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-escape-keydown": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-IM1zzRV4W3HtVgftdQiiOmA0AdJlCtMLe00FXaHwgt3rAnNsIyDqshvkIW3hj/iu5hu8ERP7KIYki6NkqDxAwQ=="],

    "@radix-ui/react-dropdown-menu": ["@radix-ui/react-dropdown-menu@2.1.15", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-menu": "2.1.15", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-mIBnOjgwo9AH3FyKaSWoSu/dYj6VdhJ7frEPiGTeXCdUFHjl9h3mFh2wwhEtINOmYXWhdpf1rY2minFsmaNgVQ=="],

    "@radix-ui/react-focus-guards": ["@radix-ui/react-focus-guards@1.1.2", "", { "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-fyjAACV62oPV925xFCrH8DR5xWhg9KYtJT4s3u54jxp+L/hbpTY2kIeEFFbFe+a/HCE94zGQMZLIpVTPVZDhaA=="],

    "@radix-ui/react-focus-scope": ["@radix-ui/react-focus-scope@1.1.7", "", { "dependencies": { "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-t2ODlkXBQyn7jkl6TNaw/MtVEVvIGelJDCG41Okq/KwUsJBwQ4XVZsHAVUkK4mBv3ewiAS3PGuUWuY2BoK4ZUw=="],

    "@radix-ui/react-hover-card": ["@radix-ui/react-hover-card@1.1.14", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-dismissable-layer": "1.1.10", "@radix-ui/react-popper": "1.2.7", "@radix-ui/react-portal": "1.1.9", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-CPYZ24Mhirm+g6D8jArmLzjYu4Eyg3TTUHswR26QgzXBHBe64BO/RHOJKzmF/Dxb4y4f9PKyJdwm/O/AhNkb+Q=="],

    "@radix-ui/react-id": ["@radix-ui/react-id@1.1.1", "", { "dependencies": { "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-kGkGegYIdQsOb4XjsfM97rXsiHaBwco+hFI66oO4s9LU+PLAC5oJ7khdOVFxkhsmlbpUqDAvXw11CluXP+jkHg=="],

    "@radix-ui/react-label": ["@radix-ui/react-label@2.1.7", "", { "dependencies": { "@radix-ui/react-primitive": "2.1.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-YT1GqPSL8kJn20djelMX7/cTRp/Y9w5IZHvfxQTVHrOqa2yMl7i/UfMqKRU5V7mEyKTrUVgJXhNQPVCG8PBLoQ=="],

    "@radix-ui/react-menu": ["@radix-ui/react-menu@2.1.15", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-collection": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-dismissable-layer": "1.1.10", "@radix-ui/react-focus-guards": "1.1.2", "@radix-ui/react-focus-scope": "1.1.7", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-popper": "1.2.7", "@radix-ui/react-portal": "1.1.9", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-roving-focus": "1.1.10", "@radix-ui/react-slot": "1.2.3", "@radix-ui/react-use-callback-ref": "1.1.1", "aria-hidden": "^1.2.4", "react-remove-scroll": "^2.6.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-tVlmA3Vb9n8SZSd+YSbuFR66l87Wiy4du+YE+0hzKQEANA+7cWKH1WgqcEX4pXqxUFQKrWQGHdvEfw00TjFiew=="],

    "@radix-ui/react-menubar": ["@radix-ui/react-menubar@1.1.15", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-collection": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-menu": "2.1.15", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-roving-focus": "1.1.10", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-Z71C7LGD+YDYo3TV81paUs8f3Zbmkvg6VLRQpKYfzioOE6n7fOhA3ApK/V/2Odolxjoc4ENk8AYCjohCNayd5A=="],

    "@radix-ui/react-navigation-menu": ["@radix-ui/react-navigation-menu@1.2.13", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-collection": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-dismissable-layer": "1.1.10", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-use-layout-effect": "1.1.1", "@radix-ui/react-use-previous": "1.1.1", "@radix-ui/react-visually-hidden": "1.2.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-WG8wWfDiJlSF5hELjwfjSGOXcBR/ZMhBFCGYe8vERpC39CQYZeq1PQ2kaYHdye3V95d06H89KGMsVCIE4LWo3g=="],

    "@radix-ui/react-popover": ["@radix-ui/react-popover@1.1.14", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-dismissable-layer": "1.1.10", "@radix-ui/react-focus-guards": "1.1.2", "@radix-ui/react-focus-scope": "1.1.7", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-popper": "1.2.7", "@radix-ui/react-portal": "1.1.9", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-slot": "1.2.3", "@radix-ui/react-use-controllable-state": "1.2.2", "aria-hidden": "^1.2.4", "react-remove-scroll": "^2.6.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-ODz16+1iIbGUfFEfKx2HTPKizg2MN39uIOV8MXeHnmdd3i/N9Wt7vU46wbHsqA0xoaQyXVcs0KIlBdOA2Y95bw=="],

    "@radix-ui/react-popper": ["@radix-ui/react-popper@1.2.7", "", { "dependencies": { "@floating-ui/react-dom": "^2.0.0", "@radix-ui/react-arrow": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-layout-effect": "1.1.1", "@radix-ui/react-use-rect": "1.1.1", "@radix-ui/react-use-size": "1.1.1", "@radix-ui/rect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-IUFAccz1JyKcf/RjB552PlWwxjeCJB8/4KxT7EhBHOJM+mN7LdW+B3kacJXILm32xawcMMjb2i0cIZpo+f9kiQ=="],

    "@radix-ui/react-portal": ["@radix-ui/react-portal@1.1.9", "", { "dependencies": { "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-bpIxvq03if6UNwXZ+HTK71JLh4APvnXntDc6XOX8UVq4XQOVl7lwok0AvIl+b8zgCw3fSaVTZMpAPPagXbKmHQ=="],

    "@radix-ui/react-presence": ["@radix-ui/react-presence@1.1.4", "", { "dependencies": { "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-ueDqRbdc4/bkaQT3GIpLQssRlFgWaL/U2z/S31qRwwLWoxHLgry3SIfCwhxeQNbirEUXFa+lq3RL3oBYXtcmIA=="],

    "@radix-ui/react-primitive": ["@radix-ui/react-primitive@2.1.3", "", { "dependencies": { "@radix-ui/react-slot": "1.2.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-m9gTwRkhy2lvCPe6QJp4d3G1TYEUHn/FzJUtq9MjH46an1wJU+GdoGC5VLof8RX8Ft/DlpshApkhswDLZzHIcQ=="],

    "@radix-ui/react-progress": ["@radix-ui/react-progress@1.1.7", "", { "dependencies": { "@radix-ui/react-context": "1.1.2", "@radix-ui/react-primitive": "2.1.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-vPdg/tF6YC/ynuBIJlk1mm7Le0VgW6ub6J2UWnTQ7/D23KXcPI1qy+0vBkgKgd38RCMJavBXpB83HPNFMTb0Fg=="],

    "@radix-ui/react-radio-group": ["@radix-ui/react-radio-group@1.3.7", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-roving-focus": "1.1.10", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-use-previous": "1.1.1", "@radix-ui/react-use-size": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-9w5XhD0KPOrm92OTTE0SysH3sYzHsSTHNvZgUBo/VZ80VdYyB5RneDbc0dKpURS24IxkoFRu/hI0i4XyfFwY6g=="],

    "@radix-ui/react-roving-focus": ["@radix-ui/react-roving-focus@1.1.10", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-collection": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-dT9aOXUen9JSsxnMPv/0VqySQf5eDQ6LCk5Sw28kamz8wSOW2bJdlX2Bg5VUIIcV+6XlHpWTIuTPCf/UNIyq8Q=="],

    "@radix-ui/react-scroll-area": ["@radix-ui/react-scroll-area@1.2.9", "", { "dependencies": { "@radix-ui/number": "1.1.1", "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-YSjEfBXnhUELsO2VzjdtYYD4CfQjvao+lhhrX5XsHD7/cyUNzljF1FHEbgTPN7LH2MClfwRMIsYlqTYpKTTe2A=="],

    "@radix-ui/react-select": ["@radix-ui/react-select@2.2.5", "", { "dependencies": { "@radix-ui/number": "1.1.1", "@radix-ui/primitive": "1.1.2", "@radix-ui/react-collection": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-dismissable-layer": "1.1.10", "@radix-ui/react-focus-guards": "1.1.2", "@radix-ui/react-focus-scope": "1.1.7", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-popper": "1.2.7", "@radix-ui/react-portal": "1.1.9", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-slot": "1.2.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-use-layout-effect": "1.1.1", "@radix-ui/react-use-previous": "1.1.1", "@radix-ui/react-visually-hidden": "1.2.3", "aria-hidden": "^1.2.4", "react-remove-scroll": "^2.6.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-HnMTdXEVuuyzx63ME0ut4+sEMYW6oouHWNGUZc7ddvUWIcfCva/AMoqEW/3wnEllriMWBa0RHspCYnfCWJQYmA=="],

    "@radix-ui/react-separator": ["@radix-ui/react-separator@1.1.7", "", { "dependencies": { "@radix-ui/react-primitive": "2.1.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-0HEb8R9E8A+jZjvmFCy/J4xhbXy3TV+9XSnGJ3KvTtjlIUy/YQ/p6UYZvi7YbeoeXdyU9+Y3scizK6hkY37baA=="],

    "@radix-ui/react-slider": ["@radix-ui/react-slider@1.3.5", "", { "dependencies": { "@radix-ui/number": "1.1.1", "@radix-ui/primitive": "1.1.2", "@radix-ui/react-collection": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-use-layout-effect": "1.1.1", "@radix-ui/react-use-previous": "1.1.1", "@radix-ui/react-use-size": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-rkfe2pU2NBAYfGaxa3Mqosi7VZEWX5CxKaanRv0vZd4Zhl9fvQrg0VM93dv3xGLGfrHuoTRF3JXH8nb9g+B3fw=="],

    "@radix-ui/react-slot": ["@radix-ui/react-slot@1.2.3", "", { "dependencies": { "@radix-ui/react-compose-refs": "1.1.2" }, "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-aeNmHnBxbi2St0au6VBVC7JXFlhLlOnvIIlePNniyUNAClzmtAUEY8/pBiK3iHjufOlwA+c20/8jngo7xcrg8A=="],

    "@radix-ui/react-switch": ["@radix-ui/react-switch@1.2.5", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-use-previous": "1.1.1", "@radix-ui/react-use-size": "1.1.1" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-5ijLkak6ZMylXsaImpZ8u4Rlf5grRmoc0p0QeX9VJtlrM4f5m3nCTX8tWga/zOA8PZYIR/t0p2Mnvd7InrJ6yQ=="],

    "@radix-ui/react-tabs": ["@radix-ui/react-tabs@1.1.12", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-roving-focus": "1.1.10", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-GTVAlRVrQrSw3cEARM0nAx73ixrWDPNZAruETn3oHCNP6SbZ/hNxdxp+u7VkIEv3/sFoLq1PfcHrl7Pnp0CDpw=="],

    "@radix-ui/react-toast": ["@radix-ui/react-toast@1.2.14", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-collection": "1.1.7", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-dismissable-layer": "1.1.10", "@radix-ui/react-portal": "1.1.9", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-callback-ref": "1.1.1", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-use-layout-effect": "1.1.1", "@radix-ui/react-visually-hidden": "1.2.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-nAP5FBxBJGQ/YfUB+r+O6USFVkWq3gAInkxyEnmvEV5jtSbfDhfa4hwX8CraCnbjMLsE7XSf/K75l9xXY7joWg=="],

    "@radix-ui/react-toggle": ["@radix-ui/react-toggle@1.1.9", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-ZoFkBBz9zv9GWer7wIjvdRxmh2wyc2oKWw6C6CseWd6/yq1DK/l5lJ+wnsmFwJZbBYqr02mrf8A2q/CVCuM3ZA=="],

    "@radix-ui/react-toggle-group": ["@radix-ui/react-toggle-group@1.1.10", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-direction": "1.1.1", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-roving-focus": "1.1.10", "@radix-ui/react-toggle": "1.1.9", "@radix-ui/react-use-controllable-state": "1.2.2" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-kiU694Km3WFLTC75DdqgM/3Jauf3rD9wxeS9XtyWFKsBUeZA337lC+6uUazT7I1DhanZ5gyD5Stf8uf2dbQxOQ=="],

    "@radix-ui/react-tooltip": ["@radix-ui/react-tooltip@1.2.7", "", { "dependencies": { "@radix-ui/primitive": "1.1.2", "@radix-ui/react-compose-refs": "1.1.2", "@radix-ui/react-context": "1.1.2", "@radix-ui/react-dismissable-layer": "1.1.10", "@radix-ui/react-id": "1.1.1", "@radix-ui/react-popper": "1.2.7", "@radix-ui/react-portal": "1.1.9", "@radix-ui/react-presence": "1.1.4", "@radix-ui/react-primitive": "2.1.3", "@radix-ui/react-slot": "1.2.3", "@radix-ui/react-use-controllable-state": "1.2.2", "@radix-ui/react-visually-hidden": "1.2.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-Ap+fNYwKTYJ9pzqW+Xe2HtMRbQ/EeWkj2qykZ6SuEV4iS/o1bZI5ssJbk4D2r8XuDuOBVz/tIx2JObtuqU+5Zw=="],

    "@radix-ui/react-use-callback-ref": ["@radix-ui/react-use-callback-ref@1.1.1", "", { "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-FkBMwD+qbGQeMu1cOHnuGB6x4yzPjho8ap5WtbEJ26umhgqVXbhekKUQO+hZEL1vU92a3wHwdp0HAcqAUF5iDg=="],

    "@radix-ui/react-use-controllable-state": ["@radix-ui/react-use-controllable-state@1.2.2", "", { "dependencies": { "@radix-ui/react-use-effect-event": "0.0.2", "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-BjasUjixPFdS+NKkypcyyN5Pmg83Olst0+c6vGov0diwTEo6mgdqVR6hxcEgFuh4QrAs7Rc+9KuGJ9TVCj0Zzg=="],

    "@radix-ui/react-use-effect-event": ["@radix-ui/react-use-effect-event@0.0.2", "", { "dependencies": { "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-Qp8WbZOBe+blgpuUT+lw2xheLP8q0oatc9UpmiemEICxGvFLYmHm9QowVZGHtJlGbS6A6yJ3iViad/2cVjnOiA=="],

    "@radix-ui/react-use-escape-keydown": ["@radix-ui/react-use-escape-keydown@1.1.1", "", { "dependencies": { "@radix-ui/react-use-callback-ref": "1.1.1" }, "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-Il0+boE7w/XebUHyBjroE+DbByORGR9KKmITzbR7MyQ4akpORYP/ZmbhAr0DG7RmmBqoOnZdy2QlvajJ2QA59g=="],

    "@radix-ui/react-use-is-hydrated": ["@radix-ui/react-use-is-hydrated@0.1.0", "", { "dependencies": { "use-sync-external-store": "^1.5.0" }, "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-U+UORVEq+cTnRIaostJv9AGdV3G6Y+zbVd+12e18jQ5A3c0xL03IhnHuiU4UV69wolOQp5GfR58NW/EgdQhwOA=="],

    "@radix-ui/react-use-layout-effect": ["@radix-ui/react-use-layout-effect@1.1.1", "", { "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-RbJRS4UWQFkzHTTwVymMTUv8EqYhOp8dOOviLj2ugtTiXRaRQS7GLGxZTLL1jWhMeoSCf5zmcZkqTl9IiYfXcQ=="],

    "@radix-ui/react-use-previous": ["@radix-ui/react-use-previous@1.1.1", "", { "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-2dHfToCj/pzca2Ck724OZ5L0EVrr3eHRNsG/b3xQJLA2hZpVCS99bLAX+hm1IHXDEnzU6by5z/5MIY794/a8NQ=="],

    "@radix-ui/react-use-rect": ["@radix-ui/react-use-rect@1.1.1", "", { "dependencies": { "@radix-ui/rect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-QTYuDesS0VtuHNNvMh+CjlKJ4LJickCMUAqjlE3+j8w+RlRpwyX3apEQKGFzbZGdo7XNG1tXa+bQqIE7HIXT2w=="],

    "@radix-ui/react-use-size": ["@radix-ui/react-use-size@1.1.1", "", { "dependencies": { "@radix-ui/react-use-layout-effect": "1.1.1" }, "peerDependencies": { "@types/react": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-ewrXRDTAqAXlkl6t/fkXWNAhFX9I+CkKlw6zjEwk86RSPKwZr3xpBRso655aqYafwtnbpHLj6toFzmd6xdVptQ=="],

    "@radix-ui/react-visually-hidden": ["@radix-ui/react-visually-hidden@1.2.3", "", { "dependencies": { "@radix-ui/react-primitive": "2.1.3" }, "peerDependencies": { "@types/react": "*", "@types/react-dom": "*", "react": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0 || ^19.0.0-rc" } }, "sha512-pzJq12tEaaIhqjbzpCuv/OypJY/BPavOofm+dbab+MHLajy277+1lLm6JFcGgF5eskJ6mquGirhXY2GD/8u8Ug=="],

    "@radix-ui/rect": ["@radix-ui/rect@1.1.1", "", {}, "sha512-HPwpGIzkl28mWyZqG52jiqDJ12waP11Pa1lGoiyUkIEuMLBP0oeK/C89esbXrxsky5we7dfd8U58nm0SgAWpVw=="],

    "@remix-run/router": ["@remix-run/router@1.23.0", "", {}, "sha512-O3rHJzAQKamUz1fvE0Qaw0xSFqsA/yafi2iqeE0pvdFtCO1viYx8QL6f3Ln/aCCTLxs68SLf0KPM9eSeM8yBnA=="],

    "@rolldown/pluginutils": ["@rolldown/pluginutils@1.0.0-beta.27", "", {}, "sha512-+d0F4MKMCbeVUJwG96uQ4SgAznZNSq93I3V+9NHA4OpvqG8mRCpGdKmK8l/dl02h2CCDHwW2FqilnTyDcAnqjA=="],

    "@rollup/rollup-android-arm-eabi": ["@rollup/rollup-android-arm-eabi@4.24.0", "", { "os": "android", "cpu": "arm" }, "sha512-Q6HJd7Y6xdB48x8ZNVDOqsbh2uByBhgK8PiQgPhwkIw/HC/YX5Ghq2mQY5sRMZWHb3VsFkWooUVOZHKr7DmDIA=="],

    "@rollup/rollup-android-arm64": ["@rollup/rollup-android-arm64@4.24.0", "", { "os": "android", "cpu": "arm64" }, "sha512-ijLnS1qFId8xhKjT81uBHuuJp2lU4x2yxa4ctFPtG+MqEE6+C5f/+X/bStmxapgmwLwiL3ih122xv8kVARNAZA=="],

    "@rollup/rollup-darwin-arm64": ["@rollup/rollup-darwin-arm64@4.24.0", "", { "os": "darwin", "cpu": "arm64" }, "sha512-bIv+X9xeSs1XCk6DVvkO+S/z8/2AMt/2lMqdQbMrmVpgFvXlmde9mLcbQpztXm1tajC3raFDqegsH18HQPMYtA=="],

    "@rollup/rollup-darwin-x64": ["@rollup/rollup-darwin-x64@4.24.0", "", { "os": "darwin", "cpu": "x64" }, "sha512-X6/nOwoFN7RT2svEQWUsW/5C/fYMBe4fnLK9DQk4SX4mgVBiTA9h64kjUYPvGQ0F/9xwJ5U5UfTbl6BEjaQdBQ=="],

    "@rollup/rollup-linux-arm-gnueabihf": ["@rollup/rollup-linux-arm-gnueabihf@4.24.0", "", { "os": "linux", "cpu": "arm" }, "sha512-0KXvIJQMOImLCVCz9uvvdPgfyWo93aHHp8ui3FrtOP57svqrF/roSSR5pjqL2hcMp0ljeGlU4q9o/rQaAQ3AYA=="],

    "@rollup/rollup-linux-arm-musleabihf": ["@rollup/rollup-linux-arm-musleabihf@4.24.0", "", { "os": "linux", "cpu": "arm" }, "sha512-it2BW6kKFVh8xk/BnHfakEeoLPv8STIISekpoF+nBgWM4d55CZKc7T4Dx1pEbTnYm/xEKMgy1MNtYuoA8RFIWw=="],

    "@rollup/rollup-linux-arm64-gnu": ["@rollup/rollup-linux-arm64-gnu@4.24.0", "", { "os": "linux", "cpu": "arm64" }, "sha512-i0xTLXjqap2eRfulFVlSnM5dEbTVque/3Pi4g2y7cxrs7+a9De42z4XxKLYJ7+OhE3IgxvfQM7vQc43bwTgPwA=="],

    "@rollup/rollup-linux-arm64-musl": ["@rollup/rollup-linux-arm64-musl@4.24.0", "", { "os": "linux", "cpu": "arm64" }, "sha512-9E6MKUJhDuDh604Qco5yP/3qn3y7SLXYuiC0Rpr89aMScS2UAmK1wHP2b7KAa1nSjWJc/f/Lc0Wl1L47qjiyQw=="],

    "@rollup/rollup-linux-powerpc64le-gnu": ["@rollup/rollup-linux-powerpc64le-gnu@4.24.0", "", { "os": "linux", "cpu": "ppc64" }, "sha512-2XFFPJ2XMEiF5Zi2EBf4h73oR1V/lycirxZxHZNc93SqDN/IWhYYSYj8I9381ikUFXZrz2v7r2tOVk2NBwxrWw=="],

    "@rollup/rollup-linux-riscv64-gnu": ["@rollup/rollup-linux-riscv64-gnu@4.24.0", "", { "os": "linux", "cpu": "none" }, "sha512-M3Dg4hlwuntUCdzU7KjYqbbd+BLq3JMAOhCKdBE3TcMGMZbKkDdJ5ivNdehOssMCIokNHFOsv7DO4rlEOfyKpg=="],

    "@rollup/rollup-linux-s390x-gnu": ["@rollup/rollup-linux-s390x-gnu@4.24.0", "", { "os": "linux", "cpu": "s390x" }, "sha512-mjBaoo4ocxJppTorZVKWFpy1bfFj9FeCMJqzlMQGjpNPY9JwQi7OuS1axzNIk0nMX6jSgy6ZURDZ2w0QW6D56g=="],

    "@rollup/rollup-linux-x64-gnu": ["@rollup/rollup-linux-x64-gnu@4.24.0", "", { "os": "linux", "cpu": "x64" }, "sha512-ZXFk7M72R0YYFN5q13niV0B7G8/5dcQ9JDp8keJSfr3GoZeXEoMHP/HlvqROA3OMbMdfr19IjCeNAnPUG93b6A=="],

    "@rollup/rollup-linux-x64-musl": ["@rollup/rollup-linux-x64-musl@4.24.0", "", { "os": "linux", "cpu": "x64" }, "sha512-w1i+L7kAXZNdYl+vFvzSZy8Y1arS7vMgIy8wusXJzRrPyof5LAb02KGr1PD2EkRcl73kHulIID0M501lN+vobQ=="],

    "@rollup/rollup-win32-arm64-msvc": ["@rollup/rollup-win32-arm64-msvc@4.24.0", "", { "os": "win32", "cpu": "arm64" }, "sha512-VXBrnPWgBpVDCVY6XF3LEW0pOU51KbaHhccHw6AS6vBWIC60eqsH19DAeeObl+g8nKAz04QFdl/Cefta0xQtUQ=="],

    "@rollup/rollup-win32-ia32-msvc": ["@rollup/rollup-win32-ia32-msvc@4.24.0", "", { "os": "win32", "cpu": "ia32" }, "sha512-xrNcGDU0OxVcPTH/8n/ShH4UevZxKIO6HJFK0e15XItZP2UcaiLFd5kiX7hJnqCbSztUF8Qot+JWBC/QXRPYWQ=="],

    "@rollup/rollup-win32-x64-msvc": ["@rollup/rollup-win32-x64-msvc@4.24.0", "", { "os": "win32", "cpu": "x64" }, "sha512-fbMkAF7fufku0N2dE5TBXcNlg0pt0cJue4xBRE2Qc5Vqikxr4VCgKj/ht6SMdFcOacVA9rqF70APJ8RN/4vMJw=="],

    "@supabase/auth-js": ["@supabase/auth-js@2.98.0", "", { "dependencies": { "tslib": "2.8.1" } }, "sha512-GBH361T0peHU91AQNzOlIrjUZw9TZbB9YDRiyFgk/3Kvr3/Z1NWUZ2athWTfHhwNNi8IrW00foyFxQD9IO/Trg=="],

    "@supabase/functions-js": ["@supabase/functions-js@2.98.0", "", { "dependencies": { "tslib": "2.8.1" } }, "sha512-N/xEyiNU5Org+d+PNCpv+TWniAXRzxIURxDYsS/m2I/sfAB/HcM9aM2Dmf5edj5oWb9GxID1OBaZ8NMmPXL+Lg=="],

    "@supabase/postgrest-js": ["@supabase/postgrest-js@2.98.0", "", { "dependencies": { "tslib": "2.8.1" } }, "sha512-v6e9WeZuJijzUut8HyXu6gMqWFepIbaeaMIm1uKzei4yLg9bC9OtEW9O14LE/9ezqNbSAnSLO5GtOLFdm7Bpkg=="],

    "@supabase/realtime-js": ["@supabase/realtime-js@2.98.0", "", { "dependencies": { "@types/phoenix": "^1.6.6", "@types/ws": "^8.18.1", "tslib": "2.8.1", "ws": "^8.18.2" } }, "sha512-rOWt28uGyFipWOSd+n0WVMr9kUXiWaa7J4hvyLCIHjRFqWm1z9CaaKAoYyfYMC1Exn3WT8WePCgiVhlAtWC2yw=="],

    "@supabase/storage-js": ["@supabase/storage-js@2.98.0", "", { "dependencies": { "iceberg-js": "^0.8.1", "tslib": "2.8.1" } }, "sha512-tzr2mG+v7ILSAZSfZMSL9OPyIH4z1ikgQ8EcQTKfMRz4EwmlFt3UnJaGzSOxyvF5b+fc9So7qdSUWTqGgeLokQ=="],

    "@supabase/supabase-js": ["@supabase/supabase-js@2.98.0", "", { "dependencies": { "@supabase/auth-js": "2.98.0", "@supabase/functions-js": "2.98.0", "@supabase/postgrest-js": "2.98.0", "@supabase/realtime-js": "2.98.0", "@supabase/storage-js": "2.98.0" } }, "sha512-Ohc97CtInLwZyiSASz7tT9/Abm/vqnIbO9REp+PivVUII8UZsuI3bngRQnYgJdFoOIwvaEII1fX1qy8x0CyNiw=="],

    "@swc/core": ["@swc/core@1.13.2", "", { "dependencies": { "@swc/counter": "^0.1.3", "@swc/types": "^0.1.23" }, "optionalDependencies": { "@swc/core-darwin-arm64": "1.13.2", "@swc/core-darwin-x64": "1.13.2", "@swc/core-linux-arm-gnueabihf": "1.13.2", "@swc/core-linux-arm64-gnu": "1.13.2", "@swc/core-linux-arm64-musl": "1.13.2", "@swc/core-linux-x64-gnu": "1.13.2", "@swc/core-linux-x64-musl": "1.13.2", "@swc/core-win32-arm64-msvc": "1.13.2", "@swc/core-win32-ia32-msvc": "1.13.2", "@swc/core-win32-x64-msvc": "1.13.2" }, "peerDependencies": { "@swc/helpers": ">=0.5.17" }, "optionalPeers": ["@swc/helpers"] }, "sha512-YWqn+0IKXDhqVLKoac4v2tV6hJqB/wOh8/Br8zjqeqBkKa77Qb0Kw2i7LOFzjFNZbZaPH6AlMGlBwNrxaauaAg=="],

    "@swc/core-darwin-arm64": ["@swc/core-darwin-arm64@1.13.2", "", { "os": "darwin", "cpu": "arm64" }, "sha512-44p7ivuLSGFJ15Vly4ivLJjg3ARo4879LtEBAabcHhSZygpmkP8eyjyWxrH3OxkY1eRZSIJe8yRZPFw4kPXFPw=="],

    "@swc/core-darwin-x64": ["@swc/core-darwin-x64@1.13.2", "", { "os": "darwin", "cpu": "x64" }, "sha512-Lb9EZi7X2XDAVmuUlBm2UvVAgSCbD3qKqDCxSI4jEOddzVOpNCnyZ/xEampdngUIyDDhhJLYU9duC+Mcsv5Y+A=="],

    "@swc/core-linux-arm-gnueabihf": ["@swc/core-linux-arm-gnueabihf@1.13.2", "", { "os": "linux", "cpu": "arm" }, "sha512-9TDe/92ee1x57x+0OqL1huG4BeljVx0nWW4QOOxp8CCK67Rpc/HHl2wciJ0Kl9Dxf2NvpNtkPvqj9+BUmM9WVA=="],

    "@swc/core-linux-arm64-gnu": ["@swc/core-linux-arm64-gnu@1.13.2", "", { "os": "linux", "cpu": "arm64" }, "sha512-KJUSl56DBk7AWMAIEcU83zl5mg3vlQYhLELhjwRFkGFMvghQvdqQ3zFOYa4TexKA7noBZa3C8fb24rI5sw9Exg=="],

    "@swc/core-linux-arm64-musl": ["@swc/core-linux-arm64-musl@1.13.2", "", { "os": "linux", "cpu": "arm64" }, "sha512-teU27iG1oyWpNh9CzcGQ48ClDRt/RCem7mYO7ehd2FY102UeTws2+OzLESS1TS1tEZipq/5xwx3FzbVgiolCiQ=="],

    "@swc/core-linux-x64-gnu": ["@swc/core-linux-x64-gnu@1.13.2", "", { "os": "linux", "cpu": "x64" }, "sha512-dRPsyPyqpLD0HMRCRpYALIh4kdOir8pPg4AhNQZLehKowigRd30RcLXGNVZcc31Ua8CiPI4QSgjOIxK+EQe4LQ=="],

    "@swc/core-linux-x64-musl": ["@swc/core-linux-x64-musl@1.13.2", "", { "os": "linux", "cpu": "x64" }, "sha512-CCxETW+KkYEQDqz1SYC15YIWYheqFC+PJVOW76Maa/8yu8Biw+HTAcblKf2isrlUtK8RvrQN94v3UXkC2NzCEw=="],

    "@swc/core-win32-arm64-msvc": ["@swc/core-win32-arm64-msvc@1.13.2", "", { "os": "win32", "cpu": "arm64" }, "sha512-Wv/QTA6PjyRLlmKcN6AmSI4jwSMRl0VTLGs57PHTqYRwwfwd7y4s2fIPJVBNbAlXd795dOEP6d/bGSQSyhOX3A=="],

    "@swc/core-win32-ia32-msvc": ["@swc/core-win32-ia32-msvc@1.13.2", "", { "os": "win32", "cpu": "ia32" }, "sha512-PuCdtNynEkUNbUXX/wsyUC+t4mamIU5y00lT5vJcAvco3/r16Iaxl5UCzhXYaWZSNVZMzPp9qN8NlSL8M5pPxw=="],

    "@swc/core-win32-x64-msvc": ["@swc/core-win32-x64-msvc@1.13.2", "", { "os": "win32", "cpu": "x64" }, "sha512-qlmMkFZJus8cYuBURx1a3YAG2G7IW44i+FEYV5/32ylKkzGNAr9tDJSA53XNnNXkAB5EXSPsOz7bn5C3JlEtdQ=="],

    "@swc/counter": ["@swc/counter@0.1.3", "", {}, "sha512-e2BR4lsJkkRlKZ/qCHPw9ZaSxc0MVUd7gtbtaB7aMvHeJVYe8sOB8DBZkP2DtISHGSku9sCK6T6cnY0CtXrOCQ=="],

    "@swc/types": ["@swc/types@0.1.23", "", { "dependencies": { "@swc/counter": "^0.1.3" } }, "sha512-u1iIVZV9Q0jxY+yM2vw/hZGDNudsN85bBpTqzAQ9rzkxW9D+e3aEM4Han+ow518gSewkXgjmEK0BD79ZcNVgPw=="],

    "@tailwindcss/typography": ["@tailwindcss/typography@0.5.16", "", { "dependencies": { "lodash.castarray": "^4.4.0", "lodash.isplainobject": "^4.0.6", "lodash.merge": "^4.6.2", "postcss-selector-parser": "6.0.10" }, "peerDependencies": { "tailwindcss": ">=3.0.0 || insiders || >=4.0.0-alpha.20 || >=4.0.0-beta.1" } }, "sha512-0wDLwCVF5V3x3b1SGXPCDcdsbDHMBe+lkFzBRaHeLvNi+nrrnZ1lA18u+OTWO8iSWU2GxUOCvlXtDuqftc1oiA=="],

    "@tanstack/query-core": ["@tanstack/query-core@5.83.0", "", {}, "sha512-0M8dA+amXUkyz5cVUm/B+zSk3xkQAcuXuz5/Q/LveT4ots2rBpPTZOzd7yJa2Utsf8D2Upl5KyjhHRY+9lB/XA=="],

    "@tanstack/react-query": ["@tanstack/react-query@5.83.0", "", { "dependencies": { "@tanstack/query-core": "5.83.0" }, "peerDependencies": { "react": "^18 || ^19" } }, "sha512-/XGYhZ3foc5H0VM2jLSD/NyBRIOK4q9kfeml4+0x2DlL6xVuAcVEW+hTlTapAmejObg0i3eNqhkr2dT+eciwoQ=="],

    "@testing-library/jest-dom": ["@testing-library/jest-dom@6.9.1", "", { "dependencies": { "@adobe/css-tools": "^4.4.0", "aria-query": "^5.0.0", "css.escape": "^1.5.1", "dom-accessibility-api": "^0.6.3", "picocolors": "^1.1.1", "redent": "^3.0.0" } }, "sha512-zIcONa+hVtVSSep9UT3jZ5rizo2BsxgyDYU7WFD5eICBE7no3881HGeb/QkGfsJs6JTkY1aQhT7rIPC7e+0nnA=="],

    "@testing-library/react": ["@testing-library/react@16.3.2", "", { "dependencies": { "@babel/runtime": "^7.12.5" }, "peerDependencies": { "@types/react": "^18.0.0 || ^19.0.0", "@types/react-dom": "^18.0.0 || ^19.0.0", "react": "^18.0.0 || ^19.0.0", "react-dom": "^18.0.0 || ^19.0.0" } }, "sha512-XU5/SytQM+ykqMnAnvB2umaJNIOsLF3PVv//1Ew4CTcpz0/BRyy/af40qqrt7SjKpDdT1saBMc42CUok5gaw+g=="],

    "@tootallnate/once": ["@tootallnate/once@2.0.0", "", {}, "sha512-XCuKFP5PS55gnMVu3dty8KPatLqUoy/ZYzDzAGCQ8JNFCkLXzmI7vNHCR+XpbZaMWQK/vQubr7PkYq8g470J/A=="],

    "@types/chai": ["@types/chai@5.2.3", "", { "dependencies": { "@types/deep-eql": "*", "assertion-error": "^2.0.1" } }, "sha512-Mw558oeA9fFbv65/y4mHtXDs9bPnFMZAL/jxdPFUpOHHIXX91mcgEHbS5Lahr+pwZFR8A7GQleRWeI6cGFC2UA=="],

    "@types/d3-array": ["@types/d3-array@3.2.1", "", {}, "sha512-Y2Jn2idRrLzUfAKV2LyRImR+y4oa2AntrgID95SHJxuMUrkNXmanDSed71sRNZysveJVt1hLLemQZIady0FpEg=="],

    "@types/d3-color": ["@types/d3-color@3.1.3", "", {}, "sha512-iO90scth9WAbmgv7ogoq57O9YpKmFBbmoEoCHDB2xMBY0+/KVrqAaCDyCE16dUspeOvIxFFRI+0sEtqDqy2b4A=="],

    "@types/d3-ease": ["@types/d3-ease@3.0.2", "", {}, "sha512-NcV1JjO5oDzoK26oMzbILE6HW7uVXOHLQvHshBUW4UMdZGfiY6v5BeQwh9a9tCzv+CeefZQHJt5SRgK154RtiA=="],

    "@types/d3-interpolate": ["@types/d3-interpolate@3.0.4", "", { "dependencies": { "@types/d3-color": "*" } }, "sha512-mgLPETlrpVV1YRJIglr4Ez47g7Yxjl1lj7YKsiMCb27VJH9W8NVM6Bb9d8kkpG/uAQS5AmbA48q2IAolKKo1MA=="],

    "@types/d3-path": ["@types/d3-path@3.1.0", "", {}, "sha512-P2dlU/q51fkOc/Gfl3Ul9kicV7l+ra934qBFXCFhrZMOL6du1TM0pm1ThYvENukyOn5h9v+yMJ9Fn5JK4QozrQ=="],

    "@types/d3-scale": ["@types/d3-scale@4.0.8", "", { "dependencies": { "@types/d3-time": "*" } }, "sha512-gkK1VVTr5iNiYJ7vWDI+yUFFlszhNMtVeneJ6lUTKPjprsvLLI9/tgEGiXJOnlINJA8FyA88gfnQsHbybVZrYQ=="],

    "@types/d3-shape": ["@types/d3-shape@3.1.6", "", { "dependencies": { "@types/d3-path": "*" } }, "sha512-5KKk5aKGu2I+O6SONMYSNflgiP0WfZIQvVUMan50wHsLG1G94JlxEVnCpQARfTtzytuY0p/9PXXZb3I7giofIA=="],

    "@types/d3-time": ["@types/d3-time@3.0.3", "", {}, "sha512-2p6olUZ4w3s+07q3Tm2dbiMZy5pCDfYwtLXXHUnVzXgQlZ/OyPtUz6OL382BkOuGlLXqfT+wqv8Fw2v8/0geBw=="],

    "@types/d3-timer": ["@types/d3-timer@3.0.2", "", {}, "sha512-Ps3T8E8dZDam6fUyNiMkekK3XUsaUEik+idO9/YjPtfj2qruF8tFBXS7XhtE4iIXBLxhmLjP3SXpLhVf21I9Lw=="],

    "@types/debug": ["@types/debug@4.1.12", "", { "dependencies": { "@types/ms": "*" } }, "sha512-vIChWdVG3LG1SMxEvI/AK+FWJthlrqlTu7fbrlywTkkaONwk/UAGaULXRlf8vkzFBLVm0zkMdCquhL5aOjhXPQ=="],

    "@types/deep-eql": ["@types/deep-eql@4.0.2", "", {}, "sha512-c9h9dVVMigMPc4bwTvC5dxqtqJZwQPePsWjPlpSOnojbor6pGqdk541lfA7AqFQr5pB1BRdq0juY9db81BwyFw=="],

    "@types/estree": ["@types/estree@1.0.6", "", {}, "sha512-AYnb1nQyY49te+VRAVgmzfcgjYS91mY5P0TKUDCLEM+gNnA+3T6rWITXRLYCpahpqSQbN5cE+gHpnPyXjHWxcw=="],

    "@types/estree-jsx": ["@types/estree-jsx@1.0.5", "", { "dependencies": { "@types/estree": "*" } }, "sha512-52CcUVNFyfb1A2ALocQw/Dd1BQFNmSdkuC3BkZ6iqhdMfQz7JWOFRuJFloOzjk+6WijU56m9oKXFAXc7o3Towg=="],

    "@types/hast": ["@types/hast@3.0.4", "", { "dependencies": { "@types/unist": "*" } }, "sha512-WPs+bbQw5aCj+x6laNGWLH3wviHtoCv/P3+otBhbOhJgG8qtpdAMlTCxLtsTWA7LH1Oh/bFCHsBn0TPS5m30EQ=="],

    "@types/json-schema": ["@types/json-schema@7.0.15", "", {}, "sha512-5+fP8P8MFNC+AyZCDxrB2pkZFPGzqQWUzpSeuuVLvm8VMcorNYavBqoFcxK8bQz4Qsbn4oUEEem4wDLfcysGHA=="],

    "@types/mdast": ["@types/mdast@4.0.4", "", { "dependencies": { "@types/unist": "*" } }, "sha512-kGaNbPh1k7AFzgpud/gMdvIm5xuECykRR+JnWKQno9TAXVa6WIVCGTPvYGekIDL4uwCZQSYbUxNBSb1aUo79oA=="],

    "@types/ms": ["@types/ms@2.1.0", "", {}, "sha512-GsCCIZDE/p3i96vtEqx+7dBUGXrc7zeSK3wwPHIaRThS+9OhWIXRqzs4d6k1SVU8g91DrNRWxWUGhp5KXQb2VA=="],

    "@types/node": ["@types/node@22.16.5", "", { "dependencies": { "undici-types": "~6.21.0" } }, "sha512-bJFoMATwIGaxxx8VJPeM8TonI8t579oRvgAuT8zFugJsJZgzqv0Fu8Mhp68iecjzG7cnN3mO2dJQ5uUM2EFrgQ=="],

    "@types/phoenix": ["@types/phoenix@1.6.7", "", {}, "sha512-oN9ive//QSBkf19rfDv45M7eZPi0eEXylht2OLEXicu5b4KoQ1OzXIw+xDSGWxSxe1JmepRR/ZH283vsu518/Q=="],

    "@types/prop-types": ["@types/prop-types@15.7.13", "", {}, "sha512-hCZTSvwbzWGvhqxp/RqVqwU999pBf2vp7hzIjiYOsl8wqOmUxkQ6ddw1cV3l8811+kdUFus/q4d1Y3E3SyEifA=="],

    "@types/react": ["@types/react@18.3.23", "", { "dependencies": { "@types/prop-types": "*", "csstype": "^3.0.2" } }, "sha512-/LDXMQh55EzZQ0uVAZmKKhfENivEvWz6E+EYzh+/MCjMhNsotd+ZHhBGIjFDTi6+fz0OhQQQLbTgdQIxxCsC0w=="],

    "@types/react-dom": ["@types/react-dom@18.3.7", "", { "peerDependencies": { "@types/react": "^18.0.0" } }, "sha512-MEe3UeoENYVFXzoXEWsvcpg6ZvlrFNlOQ7EOsvhI3CfAXwzPfO8Qwuxd40nepsYKqyyVQnTdEfv68q91yLcKrQ=="],

    "@types/unist": ["@types/unist@3.0.3", "", {}, "sha512-ko/gIFJRv177XgZsZcBwnqJN5x/Gien8qNOn0D5bQU/zAzVf9Zt3BlcUiLqhV9y4ARk0GbT3tnUiPNgnTXzc/Q=="],

    "@types/ws": ["@types/ws@8.18.1", "", { "dependencies": { "@types/node": "*" } }, "sha512-ThVF6DCVhA8kUGy+aazFQ4kXQ7E1Ty7A3ypFOe0IcJV8O/M511G99AW24irKrW56Wt44yG9+ij8FaqoBGkuBXg=="],

    "@typescript-eslint/eslint-plugin": ["@typescript-eslint/eslint-plugin@8.38.0", "", { "dependencies": { "@eslint-community/regexpp": "^4.10.0", "@typescript-eslint/scope-manager": "8.38.0", "@typescript-eslint/type-utils": "8.38.0", "@typescript-eslint/utils": "8.38.0", "@typescript-eslint/visitor-keys": "8.38.0", "graphemer": "^1.4.0", "ignore": "^7.0.0", "natural-compare": "^1.4.0", "ts-api-utils": "^2.1.0" }, "peerDependencies": { "@typescript-eslint/parser": "^8.38.0", "eslint": "^8.57.0 || ^9.0.0", "typescript": ">=4.8.4 <5.9.0" } }, "sha512-CPoznzpuAnIOl4nhj4tRr4gIPj5AfKgkiJmGQDaq+fQnRJTYlcBjbX3wbciGmpoPf8DREufuPRe1tNMZnGdanA=="],

    "@typescript-eslint/parser": ["@typescript-eslint/parser@8.38.0", "", { "dependencies": { "@typescript-eslint/scope-manager": "8.38.0", "@typescript-eslint/types": "8.38.0", "@typescript-eslint/typescript-estree": "8.38.0", "@typescript-eslint/visitor-keys": "8.38.0", "debug": "^4.3.4" }, "peerDependencies": { "eslint": "^8.57.0 || ^9.0.0", "typescript": ">=4.8.4 <5.9.0" } }, "sha512-Zhy8HCvBUEfBECzIl1PKqF4p11+d0aUJS1GeUiuqK9WmOug8YCmC4h4bjyBvMyAMI9sbRczmrYL5lKg/YMbrcQ=="],

    "@typescript-eslint/project-service": ["@typescript-eslint/project-service@8.38.0", "", { "dependencies": { "@typescript-eslint/tsconfig-utils": "^8.38.0", "@typescript-eslint/types": "^8.38.0", "debug": "^4.3.4" }, "peerDependencies": { "typescript": ">=4.8.4 <5.9.0" } }, "sha512-dbK7Jvqcb8c9QfH01YB6pORpqX1mn5gDZc9n63Ak/+jD67oWXn3Gs0M6vddAN+eDXBCS5EmNWzbSxsn9SzFWWg=="],

    "@typescript-eslint/scope-manager": ["@typescript-eslint/scope-manager@8.38.0", "", { "dependencies": { "@typescript-eslint/types": "8.38.0", "@typescript-eslint/visitor-keys": "8.38.0" } }, "sha512-WJw3AVlFFcdT9Ri1xs/lg8LwDqgekWXWhH3iAF+1ZM+QPd7oxQ6jvtW/JPwzAScxitILUIFs0/AnQ/UWHzbATQ=="],

    "@typescript-eslint/tsconfig-utils": ["@typescript-eslint/tsconfig-utils@8.38.0", "", { "peerDependencies": { "typescript": ">=4.8.4 <5.9.0" } }, "sha512-Lum9RtSE3EroKk/bYns+sPOodqb2Fv50XOl/gMviMKNvanETUuUcC9ObRbzrJ4VSd2JalPqgSAavwrPiPvnAiQ=="],

    "@typescript-eslint/type-utils": ["@typescript-eslint/type-utils@8.38.0", "", { "dependencies": { "@typescript-eslint/types": "8.38.0", "@typescript-eslint/typescript-estree": "8.38.0", "@typescript-eslint/utils": "8.38.0", "debug": "^4.3.4", "ts-api-utils": "^2.1.0" }, "peerDependencies": { "eslint": "^8.57.0 || ^9.0.0", "typescript": ">=4.8.4 <5.9.0" } }, "sha512-c7jAvGEZVf0ao2z+nnz8BUaHZD09Agbh+DY7qvBQqLiz8uJzRgVPj5YvOh8I8uEiH8oIUGIfHzMwUcGVco/SJg=="],

    "@typescript-eslint/types": ["@typescript-eslint/types@8.38.0", "", {}, "sha512-wzkUfX3plUqij4YwWaJyqhiPE5UCRVlFpKn1oCRn2O1bJ592XxWJj8ROQ3JD5MYXLORW84063z3tZTb/cs4Tyw=="],

    "@typescript-eslint/typescript-estree": ["@typescript-eslint/typescript-estree@8.38.0", "", { "dependencies": { "@typescript-eslint/project-service": "8.38.0", "@typescript-eslint/tsconfig-utils": "8.38.0", "@typescript-eslint/types": "8.38.0", "@typescript-eslint/visitor-keys": "8.38.0", "debug": "^4.3.4", "fast-glob": "^3.3.2", "is-glob": "^4.0.3", "minimatch": "^9.0.4", "semver": "^7.6.0", "ts-api-utils": "^2.1.0" }, "peerDependencies": { "typescript": ">=4.8.4 <5.9.0" } }, "sha512-fooELKcAKzxux6fA6pxOflpNS0jc+nOQEEOipXFNjSlBS6fqrJOVY/whSn70SScHrcJ2LDsxWrneFoWYSVfqhQ=="],

    "@typescript-eslint/utils": ["@typescript-eslint/utils@8.38.0", "", { "dependencies": { "@eslint-community/eslint-utils": "^4.7.0", "@typescript-eslint/scope-manager": "8.38.0", "@typescript-eslint/types": "8.38.0", "@typescript-eslint/typescript-estree": "8.38.0" }, "peerDependencies": { "eslint": "^8.57.0 || ^9.0.0", "typescript": ">=4.8.4 <5.9.0" } }, "sha512-hHcMA86Hgt+ijJlrD8fX0j1j8w4C92zue/8LOPAFioIno+W0+L7KqE8QZKCcPGc/92Vs9x36w/4MPTJhqXdyvg=="],

    "@typescript-eslint/visitor-keys": ["@typescript-eslint/visitor-keys@8.38.0", "", { "dependencies": { "@typescript-eslint/types": "8.38.0", "eslint-visitor-keys": "^4.2.1" } }, "sha512-pWrTcoFNWuwHlA9CvlfSsGWs14JxfN1TH25zM5L7o0pRLhsoZkDnTsXfQRJBEWJoV5DL0jf+Z+sxiud+K0mq1g=="],

    "@ungap/structured-clone": ["@ungap/structured-clone@1.3.0", "", {}, "sha512-WmoN8qaIAo7WTYWbAZuG8PYEhn5fkz7dZrqTBZ7dtt//lL2Gwms1IcnQ5yHqjDfX8Ft5j4YzDM23f87zBfDe9g=="],

    "@vitejs/plugin-react-swc": ["@vitejs/plugin-react-swc@3.11.0", "", { "dependencies": { "@rolldown/pluginutils": "1.0.0-beta.27", "@swc/core": "^1.12.11" }, "peerDependencies": { "vite": "^4 || ^5 || ^6 || ^7" } }, "sha512-YTJCGFdNMHCMfjODYtxRNVAYmTWQ1Lb8PulP/2/f/oEEtglw8oKxKIZmmRkyXrVrHfsKOaVkAc3NT9/dMutO5w=="],

    "@vitest/expect": ["@vitest/expect@3.2.4", "", { "dependencies": { "@types/chai": "^5.2.2", "@vitest/spy": "3.2.4", "@vitest/utils": "3.2.4", "chai": "^5.2.0", "tinyrainbow": "^2.0.0" } }, "sha512-Io0yyORnB6sikFlt8QW5K7slY4OjqNX9jmJQ02QDda8lyM6B5oNgVWoSoKPac8/kgnCUzuHQKrSLtu/uOqqrig=="],

    "@vitest/mocker": ["@vitest/mocker@3.2.4", "", { "dependencies": { "@vitest/spy": "3.2.4", "estree-walker": "^3.0.3", "magic-string": "^0.30.17" }, "peerDependencies": { "msw": "^2.4.9", "vite": "^5.0.0 || ^6.0.0 || ^7.0.0-0" }, "optionalPeers": ["msw"] }, "sha512-46ryTE9RZO/rfDd7pEqFl7etuyzekzEhUbTW3BvmeO/BcCMEgq59BKhek3dXDWgAj4oMK6OZi+vRr1wPW6qjEQ=="],

    "@vitest/pretty-format": ["@vitest/pretty-format@3.2.4", "", { "dependencies": { "tinyrainbow": "^2.0.0" } }, "sha512-IVNZik8IVRJRTr9fxlitMKeJeXFFFN0JaB9PHPGQ8NKQbGpfjlTx9zO4RefN8gp7eqjNy8nyK3NZmBzOPeIxtA=="],

    "@vitest/runner": ["@vitest/runner@3.2.4", "", { "dependencies": { "@vitest/utils": "3.2.4", "pathe": "^2.0.3", "strip-literal": "^3.0.0" } }, "sha512-oukfKT9Mk41LreEW09vt45f8wx7DordoWUZMYdY/cyAk7w5TWkTRCNZYF7sX7n2wB7jyGAl74OxgwhPgKaqDMQ=="],

    "@vitest/snapshot": ["@vitest/snapshot@3.2.4", "", { "dependencies": { "@vitest/pretty-format": "3.2.4", "magic-string": "^0.30.17", "pathe": "^2.0.3" } }, "sha512-dEYtS7qQP2CjU27QBC5oUOxLE/v5eLkGqPE0ZKEIDGMs4vKWe7IjgLOeauHsR0D5YuuycGRO5oSRXnwnmA78fQ=="],

    "@vitest/spy": ["@vitest/spy@3.2.4", "", { "dependencies": { "tinyspy": "^4.0.3" } }, "sha512-vAfasCOe6AIK70iP5UD11Ac4siNUNJ9i/9PZ3NKx07sG6sUxeag1LWdNrMWeKKYBLlzuK+Gn65Yd5nyL6ds+nw=="],

    "@vitest/utils": ["@vitest/utils@3.2.4", "", { "dependencies": { "@vitest/pretty-format": "3.2.4", "loupe": "^3.1.4", "tinyrainbow": "^2.0.0" } }, "sha512-fB2V0JFrQSMsCo9HiSq3Ezpdv4iYaXRG1Sx8edX3MwxfyNn83mKiGzOcH+Fkxt4MHxr3y42fQi1oeAInqgX2QA=="],

    "abab": ["abab@2.0.6", "", {}, "sha512-j2afSsaIENvHZN2B8GOpF566vZ5WVk5opAiMTvWgaQT8DkbOqsTfvNAvHoRGU2zzP8cPoqys+xHTRDWW8L+/BA=="],

    "acorn": ["acorn@8.15.0", "", { "bin": "bin/acorn" }, "sha512-NZyJarBfL7nWwIq+FDL6Zp/yHEhePMNnnJ0y3qfieCrmNvYct8uvtiV41UvlSe6apAfk0fY1FbWx+NwfmpvtTg=="],

    "acorn-globals": ["acorn-globals@7.0.1", "", { "dependencies": { "acorn": "^8.1.0", "acorn-walk": "^8.0.2" } }, "sha512-umOSDSDrfHbTNPuNpC2NSnnA3LUrqpevPb4T9jRx4MagXNS0rs+gwiTcAvqCRmsD6utzsrzNt+ebm00SNWiC3Q=="],

    "acorn-jsx": ["acorn-jsx@5.3.2", "", { "peerDependencies": { "acorn": "^6.0.0 || ^7.0.0 || ^8.0.0" } }, "sha512-rq9s+JNhf0IChjtDXxllJ7g41oZk5SlXtp0LHwyA5cejwn7vKmKp4pPri6YEePv2PU65sAsegbXtIinmDFDXgQ=="],

    "acorn-walk": ["acorn-walk@8.3.5", "", { "dependencies": { "acorn": "^8.11.0" } }, "sha512-HEHNfbars9v4pgpW6SO1KSPkfoS0xVOM/9UzkJltjlsHZmJasxg8aXkuZa7SMf8vKGIBhpUsPluQSqhJFCqebw=="],

    "agent-base": ["agent-base@6.0.2", "", { "dependencies": { "debug": "4" } }, "sha512-RZNwNclF7+MS/8bDg70amg32dyeZGZxiDuQmZxKLAlQjr3jGyLx+4Kkk58UO7D2QdgFIQCovuSuZESne6RG6XQ=="],

    "ajv": ["ajv@6.12.6", "", { "dependencies": { "fast-deep-equal": "^3.1.1", "fast-json-stable-stringify": "^2.0.0", "json-schema-traverse": "^0.4.1", "uri-js": "^4.2.2" } }, "sha512-j3fVLgvTo527anyYyJOGTYJbG+vnnQYvE0m5mmkc1TK+nxAppkCLMIL0aZ4dblVCNoGShhm+kzE4ZUykBoMg4g=="],

    "ansi-regex": ["ansi-regex@6.1.0", "", {}, "sha512-7HSX4QQb4CspciLpVFwyRe79O3xsIZDDLER21kERQ71oaPodF8jL725AgJMFAYbooIqolJoRLuM81SpeUkpkvA=="],

    "ansi-styles": ["ansi-styles@4.3.0", "", { "dependencies": { "color-convert": "^2.0.1" } }, "sha512-zbB9rCJAT1rbjiVDb2hqKFHNYLxgtk8NURxZ3IZwD3F6NtxbXZQCnnSi1Lkx+IDohdPlFp222wVALIheZJQSEg=="],

    "any-promise": ["any-promise@1.3.0", "", {}, "sha512-7UvmKalWRt1wgjL1RrGxoSJW/0QZFIegpeGvZG9kjp8vrRu55XTHbwnqq2GpXm9uLbcuhxm3IqX9OB4MZR1b2A=="],

    "anymatch": ["anymatch@3.1.3", "", { "dependencies": { "normalize-path": "^3.0.0", "picomatch": "^2.0.4" } }, "sha512-KMReFUr0B4t+D+OBkjR3KYqvocp2XaSzO55UcB6mgQMd3KbcE+mWTyvVV7D/zsdEbNnV6acZUutkiHQXvTr1Rw=="],

    "arg": ["arg@5.0.2", "", {}, "sha512-PYjyFOLKQ9y57JvQ6QLo8dAgNqswh8M1RMJYdQduT6xbWSgK36P/Z/v+p888pM69jMMfS8Xd8F6I1kQ/I9HUGg=="],

    "argparse": ["argparse@2.0.1", "", {}, "sha512-8+9WqebbFzpX9OR+Wa6O29asIogeRMzcGtAINdpMHHyAg10f05aSFVBbcEqGf/PXw1EjAZ+q2/bEBg3DvurK3Q=="],

    "aria-hidden": ["aria-hidden@1.2.4", "", { "dependencies": { "tslib": "^2.0.0" } }, "sha512-y+CcFFwelSXpLZk/7fMB2mUbGtX9lKycf1MWJ7CaTIERyitVlyQx6C+sxcROU2BAJ24OiZyK+8wj2i8AlBoS3A=="],

    "aria-query": ["aria-query@5.3.2", "", {}, "sha512-COROpnaoap1E2F000S62r6A60uHZnmlvomhfyT2DlTcrY1OrBKn2UhH7qn5wTC9zMvD0AY7csdPSNwKP+7WiQw=="],

    "assertion-error": ["assertion-error@2.0.1", "", {}, "sha512-Izi8RQcffqCeNVgFigKli1ssklIbpHnCYc6AknXGYoB6grJqyeby7jv12JUQgmTAnIDnbck1uxksT4dzN3PWBA=="],

    "asynckit": ["asynckit@0.4.0", "", {}, "sha512-Oei9OH4tRh0YqU3GxhX79dM/mwVgvbZJaSNaRk+bshkj0S5cfHcgYakreBjrHwatXKbz+IoIdYLxrKim2MjW0Q=="],

    "autoprefixer": ["autoprefixer@10.4.21", "", { "dependencies": { "browserslist": "^4.24.4", "caniuse-lite": "^1.0.30001702", "fraction.js": "^4.3.7", "normalize-range": "^0.1.2", "picocolors": "^1.1.1", "postcss-value-parser": "^4.2.0" }, "peerDependencies": { "postcss": "^8.1.0" }, "bin": "bin/autoprefixer" }, "sha512-O+A6LWV5LDHSJD3LjHYoNi4VLsj/Whi7k6zG12xTYaU4cQ8oxQGckXNX8cRHK5yOZ/ppVHe0ZBXGzSV9jXdVbQ=="],

    "bail": ["bail@2.0.2", "", {}, "sha512-0xO6mYd7JB2YesxDKplafRpsiOzPt9V02ddPCLbY1xYGPOX24NTyN50qnUxgCPcSoYMhKpAuBTjQoRZCAkUDRw=="],

    "balanced-match": ["balanced-match@1.0.2", "", {}, "sha512-3oSeUO0TMV67hN1AmbXsK4yaqU7tjiHlbxRDZOpH0KW9+CeX4bRAaX0Anxt0tx2MrpRpWwQaPwIlISEJhYU5Pw=="],

    "binary-extensions": ["binary-extensions@2.3.0", "", {}, "sha512-Ceh+7ox5qe7LJuLHoY0feh3pHuUDHAcRUeyL2VYghZwfpkNIy/+8Ocg0a3UuSoYzavmylwuLWQOf3hl0jjMMIw=="],

    "brace-expansion": ["brace-expansion@1.1.12", "", { "dependencies": { "balanced-match": "^1.0.0", "concat-map": "0.0.1" } }, "sha512-9T9UjW3r0UW5c1Q7GTwllptXwhvYmEzFhzMfZ9H7FQWt+uZePjZPjBP/W1ZEyZ1twGWom5/56TF4lPcqjnDHcg=="],

    "braces": ["braces@3.0.3", "", { "dependencies": { "fill-range": "^7.1.1" } }, "sha512-yQbXgO/OSZVD2IsiLlro+7Hf6Q18EJrKSEsdoMzKePKXct3gvD8oLcOQdIzGupr5Fj+EDe8gO/lxc1BzfMpxvA=="],

    "browserslist": ["browserslist@4.25.1", "", { "dependencies": { "caniuse-lite": "^1.0.30001726", "electron-to-chromium": "^1.5.173", "node-releases": "^2.0.19", "update-browserslist-db": "^1.1.3" }, "bin": "cli.js" }, "sha512-KGj0KoOMXLpSNkkEI6Z6mShmQy0bc1I+T7K9N81k4WWMrfz+6fQ6es80B/YLAeRoKvjYE1YSHHOW1qe9xIVzHw=="],

    "cac": ["cac@6.7.14", "", {}, "sha512-b6Ilus+c3RrdDk+JhLKUAQfzzgLEPy6wcXqS7f/xe1EETvsDP6GORG7SFuOs6cID5YkqchW/LXZbX5bc8j7ZcQ=="],

    "call-bind-apply-helpers": ["call-bind-apply-helpers@1.0.2", "", { "dependencies": { "es-errors": "^1.3.0", "function-bind": "^1.1.2" } }, "sha512-Sp1ablJ0ivDkSzjcaJdxEunN5/XvksFJ2sMBFfq6x0ryhQV/2b/KwFe21cMpmHtPOSij8K99/wSfoEuTObmuMQ=="],

    "callsites": ["callsites@3.1.0", "", {}, "sha512-P8BjAsXvZS+VIDUI11hHCQEv74YT67YUi5JJFNWIqL235sBmjX4+qx9Muvls5ivyNENctx46xQLQ3aTuE7ssaQ=="],

    "camelcase-css": ["camelcase-css@2.0.1", "", {}, "sha512-QOSvevhslijgYwRx6Rv7zKdMF8lbRmx+uQGx2+vDc+KI/eBnsy9kit5aj23AgGu3pa4t9AgwbnXWqS+iOY+2aA=="],

    "caniuse-lite": ["caniuse-lite@1.0.30001727", "", {}, "sha512-pB68nIHmbN6L/4C6MH1DokyR3bYqFwjaSs/sWDHGj4CTcFtQUQMuJftVwWkXq7mNWOybD3KhUv3oWHoGxgP14Q=="],

    "ccount": ["ccount@2.0.1", "", {}, "sha512-eyrF0jiFpY+3drT6383f1qhkbGsLSifNAjA61IUjZjmLCWjItY6LB9ft9YhoDgwfmclB2zhu51Lc7+95b8NRAg=="],

    "chai": ["chai@5.3.3", "", { "dependencies": { "assertion-error": "^2.0.1", "check-error": "^2.1.1", "deep-eql": "^5.0.1", "loupe": "^3.1.0", "pathval": "^2.0.0" } }, "sha512-4zNhdJD/iOjSH0A05ea+Ke6MU5mmpQcbQsSOkgdaUMJ9zTlDTD/GYlwohmIE2u0gaxHYiVHEn1Fw9mZ/ktJWgw=="],

    "chalk": ["chalk@4.1.2", "", { "dependencies": { "ansi-styles": "^4.1.0", "supports-color": "^7.1.0" } }, "sha512-oKnbhFyRIXpUuez8iBMmyEa4nbj4IOQyuhc/wy9kY7/WVPcwIO9VA668Pu8RkO7+0G76SLROeyw9CpQ061i4mA=="],

    "character-entities": ["character-entities@2.0.2", "", {}, "sha512-shx7oQ0Awen/BRIdkjkvz54PnEEI/EjwXDSIZp86/KKdbafHh1Df/RYGBhn4hbe2+uKC9FnT5UCEdyPz3ai9hQ=="],

    "character-entities-html4": ["character-entities-html4@2.1.0", "", {}, "sha512-1v7fgQRj6hnSwFpq1Eu0ynr/CDEw0rXo2B61qXrLNdHZmPKgb7fqS1a2JwF0rISo9q77jDI8VMEHoApn8qDoZA=="],

    "character-entities-legacy": ["character-entities-legacy@3.0.0", "", {}, "sha512-RpPp0asT/6ufRm//AJVwpViZbGM/MkjQFxJccQRHmISF/22NBtsHqAWmL+/pmkPWoIUJdWyeVleTl1wydHATVQ=="],

    "character-reference-invalid": ["character-reference-invalid@2.0.1", "", {}, "sha512-iBZ4F4wRbyORVsu0jPV7gXkOsGYjGHPmAyv+HiHG8gi5PtC9KI2j1+v8/tlibRvjoWX027ypmG/n0HtO5t7unw=="],

    "check-error": ["check-error@2.1.3", "", {}, "sha512-PAJdDJusoxnwm1VwW07VWwUN1sl7smmC3OKggvndJFadxxDRyFJBX/ggnu/KE4kQAB7a3Dp8f/YXC1FlUprWmA=="],

    "chokidar": ["chokidar@3.6.0", "", { "dependencies": { "anymatch": "~3.1.2", "braces": "~3.0.2", "glob-parent": "~5.1.2", "is-binary-path": "~2.1.0", "is-glob": "~4.0.1", "normalize-path": "~3.0.0", "readdirp": "~3.6.0" }, "optionalDependencies": { "fsevents": "~2.3.2" } }, "sha512-7VT13fmjotKpGipCW9JEQAusEPE+Ei8nl6/g4FBAmIm0GOOLMua9NDDo/DWp0ZAxCr3cPq5ZpBqmPAQgDda2Pw=="],

    "class-variance-authority": ["class-variance-authority@0.7.1", "", { "dependencies": { "clsx": "^2.1.1" } }, "sha512-Ka+9Trutv7G8M6WT6SeiRWz792K5qEqIGEGzXKhAE6xOWAY6pPH8U+9IY3oCMv6kqTmLsv7Xh/2w2RigkePMsg=="],

    "clsx": ["clsx@2.1.1", "", {}, "sha512-eYm0QWBtUrBWZWG0d386OGAw16Z995PiOVo2B7bjWSbHedGl5e0ZWaq65kOGgUSNesEIDkB9ISbTg/JK9dhCZA=="],

    "cmdk": ["cmdk@1.1.1", "", { "dependencies": { "@radix-ui/react-compose-refs": "^1.1.1", "@radix-ui/react-dialog": "^1.1.6", "@radix-ui/react-id": "^1.1.0", "@radix-ui/react-primitive": "^2.0.2" }, "peerDependencies": { "react": "^18 || ^19 || ^19.0.0-rc", "react-dom": "^18 || ^19 || ^19.0.0-rc" } }, "sha512-Vsv7kFaXm+ptHDMZ7izaRsP70GgrW9NBNGswt9OZaVBLlE0SNpDq8eu/VGXyF9r7M0azK3Wy7OlYXsuyYLFzHg=="],

    "color-convert": ["color-convert@2.0.1", "", { "dependencies": { "color-name": "~1.1.4" } }, "sha512-RRECPsj7iu/xb5oKYcsFHSppFNnsj/52OVTRKb4zP5onXwVF3zVmmToNcOfGC+CRDpfK/U584fMg38ZHCaElKQ=="],

    "color-name": ["color-name@1.1.4", "", {}, "sha512-dOy+3AuW3a2wNbZHIuMZpTcgjGuLU/uBL/ubcZF9OXbDo8ff4O8yVp5Bf0efS8uEoYo5q4Fx7dY9OgQGXgAsQA=="],

    "combined-stream": ["combined-stream@1.0.8", "", { "dependencies": { "delayed-stream": "~1.0.0" } }, "sha512-FQN4MRfuJeHf7cBbBMJFXhKSDq+2kAArBlmRBvcvFE5BB1HZKXtSFASDhdlz9zOYwxh8lDdnvmMOe/+5cdoEdg=="],

    "comma-separated-tokens": ["comma-separated-tokens@2.0.3", "", {}, "sha512-Fu4hJdvzeylCfQPp9SGWidpzrMs7tTrlu6Vb8XGaRGck8QSNZJJp538Wrb60Lax4fPwR64ViY468OIUTbRlGZg=="],

    "commander": ["commander@4.1.1", "", {}, "sha512-NOKm8xhkzAjzFx8B2v5OAHT+u5pRQc2UCa2Vq9jYL/31o2wi9mxBA7LIFs3sV5VSC49z6pEhfbMULvShKj26WA=="],

    "concat-map": ["concat-map@0.0.1", "", {}, "sha512-/Srv4dswyQNBfohGpz9o6Yb3Gz3SrUDqBH5rTuhGR7ahtlbYKnVxw2bCFMRljaA7EXHaXZ8wsHdodFvbkhKmqg=="],

    "cross-spawn": ["cross-spawn@7.0.6", "", { "dependencies": { "path-key": "^3.1.0", "shebang-command": "^2.0.0", "which": "^2.0.1" } }, "sha512-uV2QOWP2nWzsy2aMp8aRibhi9dlzF5Hgh5SHaB9OiTGEyDTiJJyx0uy51QXdyWbtAHNua4XJzUKca3OzKUd3vA=="],

    "css.escape": ["css.escape@1.5.1", "", {}, "sha512-YUifsXXuknHlUsmlgyY0PKzgPOr7/FjCePfHNt0jxm83wHZi44VDMQ7/fGNkjY3/jV1MC+1CmZbaHzugyeRtpg=="],

    "cssesc": ["cssesc@3.0.0", "", { "bin": "bin/cssesc" }, "sha512-/Tb/JcjK111nNScGob5MNtsntNM1aCNUDipB/TkwZFhyDrrE47SOx/18wF2bbjgc3ZzCSKW1T5nt5EbFoAz/Vg=="],

    "cssom": ["cssom@0.5.0", "", {}, "sha512-iKuQcq+NdHqlAcwUY0o/HL69XQrUaQdMjmStJ8JFmUaiiQErlhrmuigkg/CU4E2J0IyUKUrMAgl36TvN67MqTw=="],

    "cssstyle": ["cssstyle@2.3.0", "", { "dependencies": { "cssom": "~0.3.6" } }, "sha512-AZL67abkUzIuvcHqk7c09cezpGNcxUxU4Ioi/05xHk4DQeTkWmGYftIE6ctU6AEt+Gn4n1lDStOtj7FKycP71A=="],

    "csstype": ["csstype@3.1.3", "", {}, "sha512-M1uQkMl8rQK/szD0LNhtqxIPLpimGm8sOBwU7lLnCpSbTyY3yeU1Vc7l4KT5zT4s/yOxHH5O7tIuuLOCnLADRw=="],

    "d3-array": ["d3-array@3.2.4", "", { "dependencies": { "internmap": "1 - 2" } }, "sha512-tdQAmyA18i4J7wprpYq8ClcxZy3SC31QMeByyCFyRt7BVHdREQZ5lpzoe5mFEYZUWe+oq8HBvk9JjpibyEV4Jg=="],

    "d3-color": ["d3-color@3.1.0", "", {}, "sha512-zg/chbXyeBtMQ1LbD/WSoW2DpC3I0mpmPdW+ynRTj/x2DAWYrIY7qeZIHidozwV24m4iavr15lNwIwLxRmOxhA=="],

    "d3-ease": ["d3-ease@3.0.1", "", {}, "sha512-wR/XK3D3XcLIZwpbvQwQ5fK+8Ykds1ip7A2Txe0yxncXSdq1L9skcG7blcedkOX+ZcgxGAmLX1FrRGbADwzi0w=="],

    "d3-format": ["d3-format@3.1.0", "", {}, "sha512-YyUI6AEuY/Wpt8KWLgZHsIU86atmikuoOmCfommt0LYHiQSPjvX2AcFc38PX0CBpr2RCyZhjex+NS/LPOv6YqA=="],

    "d3-interpolate": ["d3-interpolate@3.0.1", "", { "dependencies": { "d3-color": "1 - 3" } }, "sha512-3bYs1rOD33uo8aqJfKP3JWPAibgw8Zm2+L9vBKEHJ2Rg+viTR7o5Mmv5mZcieN+FRYaAOWX5SJATX6k1PWz72g=="],

    "d3-path": ["d3-path@3.1.0", "", {}, "sha512-p3KP5HCf/bvjBSSKuXid6Zqijx7wIfNW+J/maPs+iwR35at5JCbLUT0LzF1cnjbCHWhqzQTIN2Jpe8pRebIEFQ=="],

    "d3-scale": ["d3-scale@4.0.2", "", { "dependencies": { "d3-array": "2.10.0 - 3", "d3-format": "1 - 3", "d3-interpolate": "1.2.0 - 3", "d3-time": "2.1.1 - 3", "d3-time-format": "2 - 4" } }, "sha512-GZW464g1SH7ag3Y7hXjf8RoUuAFIqklOAq3MRl4OaWabTFJY9PN/E1YklhXLh+OQ3fM9yS2nOkCoS+WLZ6kvxQ=="],

    "d3-shape": ["d3-shape@3.2.0", "", { "dependencies": { "d3-path": "^3.1.0" } }, "sha512-SaLBuwGm3MOViRq2ABk3eLoxwZELpH6zhl3FbAoJ7Vm1gofKx6El1Ib5z23NUEhF9AsGl7y+dzLe5Cw2AArGTA=="],

    "d3-time": ["d3-time@3.1.0", "", { "dependencies": { "d3-array": "2 - 3" } }, "sha512-VqKjzBLejbSMT4IgbmVgDjpkYrNWUYJnbCGo874u7MMKIWsILRX+OpX/gTk8MqjpT1A/c6HY2dCA77ZN0lkQ2Q=="],

    "d3-time-format": ["d3-time-format@4.1.0", "", { "dependencies": { "d3-time": "1 - 3" } }, "sha512-dJxPBlzC7NugB2PDLwo9Q8JiTR3M3e4/XANkreKSUxF8vvXKqm1Yfq4Q5dl8budlunRVlUUaDUgFt7eA8D6NLg=="],

    "d3-timer": ["d3-timer@3.0.1", "", {}, "sha512-ndfJ/JxxMd3nw31uyKoY2naivF+r29V+Lc0svZxe1JvvIRmi8hUsrMvdOwgS1o6uBHmiz91geQ0ylPP0aj1VUA=="],

    "data-urls": ["data-urls@3.0.2", "", { "dependencies": { "abab": "^2.0.6", "whatwg-mimetype": "^3.0.0", "whatwg-url": "^11.0.0" } }, "sha512-Jy/tj3ldjZJo63sVAvg6LHt2mHvl4V6AgRAmNDtLdm7faqtsx+aJG42rsyCo9JCoRVKwPFzKlIPx3DIibwSIaQ=="],

    "date-fns": ["date-fns@3.6.0", "", {}, "sha512-fRHTG8g/Gif+kSh50gaGEdToemgfj74aRX3swtiouboip5JDLAyDE9F11nHMIcvOaXeOC6D7SpNhi7uFyB7Uww=="],

    "debug": ["debug@4.4.3", "", { "dependencies": { "ms": "^2.1.3" } }, "sha512-RGwwWnwQvkVfavKVt22FGLw+xYSdzARwm0ru6DhTVA3umU5hZc28V3kO4stgYryrTlLpuvgI9GiijltAjNbcqA=="],

    "decimal.js": ["decimal.js@10.6.0", "", {}, "sha512-YpgQiITW3JXGntzdUmyUR1V812Hn8T1YVXhCu+wO3OpS4eU9l4YdD3qjyiKdV6mvV29zapkMeD390UVEf2lkUg=="],

    "decimal.js-light": ["decimal.js-light@2.5.1", "", {}, "sha512-qIMFpTMZmny+MMIitAB6D7iVPEorVw6YQRWkvarTkT4tBeSLLiHzcwj6q0MmYSFCiVpiqPJTJEYIrpcPzVEIvg=="],

    "decode-named-character-reference": ["decode-named-character-reference@1.3.0", "", { "dependencies": { "character-entities": "^2.0.0" } }, "sha512-GtpQYB283KrPp6nRw50q3U9/VfOutZOe103qlN7BPP6Ad27xYnOIWv4lPzo8HCAL+mMZofJ9KEy30fq6MfaK6Q=="],

    "deep-eql": ["deep-eql@5.0.2", "", {}, "sha512-h5k/5U50IJJFpzfL6nO9jaaumfjO/f2NjK/oYB2Djzm4p9L+3T9qWpZqZ2hAbLPuuYq9wrU08WQyBTL5GbPk5Q=="],

    "deep-is": ["deep-is@0.1.4", "", {}, "sha512-oIPzksmTg4/MriiaYGO+okXDT7ztn/w3Eptv/+gSIdMdKsJo0u4CfYNFJPy+4SKMuCqGw2wxnA+URMg3t8a/bQ=="],

    "delayed-stream": ["delayed-stream@1.0.0", "", {}, "sha512-ZySD7Nf91aLB0RxL4KGrKHBXl7Eds1DAmEdcoVawXnLD7SDhpNgtuII2aAkg7a7QS41jxPSZ17p4VdGnMHk3MQ=="],

    "dequal": ["dequal@2.0.3", "", {}, "sha512-0je+qPKHEMohvfRTCEo3CrPG6cAzAYgmzKyxRiYSSDkS6eGJdyVJm7WaYA5ECaAD9wLB2T4EEeymA5aFVcYXCA=="],

    "detect-node-es": ["detect-node-es@1.1.0", "", {}, "sha512-ypdmJU/TbBby2Dxibuv7ZLW3Bs1QEmM7nHjEANfohJLvE0XVujisn1qPJcZxg+qDucsr+bP6fLD1rPS3AhJ7EQ=="],

    "devlop": ["devlop@1.1.0", "", { "dependencies": { "dequal": "^2.0.0" } }, "sha512-RWmIqhcFf1lRYBvNmr7qTNuyCt/7/ns2jbpp1+PalgE/rDQcBT0fioSMUpJ93irlUhC5hrg4cYqe6U+0ImW0rA=="],

    "didyoumean": ["didyoumean@1.2.2", "", {}, "sha512-gxtyfqMg7GKyhQmb056K7M3xszy/myH8w+B4RT+QXBQsvAOdc3XymqDDPHx1BgPgsdAA5SIifona89YtRATDzw=="],

    "dlv": ["dlv@1.1.3", "", {}, "sha512-+HlytyjlPKnIG8XuRG8WvmBP8xs8P71y+SKKS6ZXWoEgLuePxtDoUEiH7WkdePWrQ5JBpE6aoVqfZfJUQkjXwA=="],

    "dom-accessibility-api": ["dom-accessibility-api@0.6.3", "", {}, "sha512-7ZgogeTnjuHbo+ct10G9Ffp0mif17idi0IyWNVA/wcwcm7NPOD/WEHVP3n7n3MhXqxoIYm8d6MuZohYWIZ4T3w=="],

    "dom-helpers": ["dom-helpers@5.2.1", "", { "dependencies": { "@babel/runtime": "^7.8.7", "csstype": "^3.0.2" } }, "sha512-nRCa7CK3VTrM2NmGkIy4cbK7IZlgBE/PYMn55rrXefr5xXDP0LdtfPnblFDoVdcAfslJ7or6iqAUnx0CCGIWQA=="],

    "domexception": ["domexception@4.0.0", "", { "dependencies": { "webidl-conversions": "^7.0.0" } }, "sha512-A2is4PLG+eeSfoTMA95/s4pvAoSo2mKtiM5jlHkAVewmiO8ISFTFKZjH7UAM1Atli/OT/7JHOrJRJiMKUZKYBw=="],

    "dunder-proto": ["dunder-proto@1.0.1", "", { "dependencies": { "call-bind-apply-helpers": "^1.0.1", "es-errors": "^1.3.0", "gopd": "^1.2.0" } }, "sha512-KIN/nDJBQRcXw0MLVhZE9iQHmG68qAVIBg9CqmUYjmQIhgij9U5MFvrqkUL5FbtyyzZuOeOt0zdeRe4UY7ct+A=="],

    "eastasianwidth": ["eastasianwidth@0.2.0", "", {}, "sha512-I88TYZWc9XiYHRQ4/3c5rjjfgkjhLyW2luGIheGERbNQ6OY7yTybanSpDXZa8y7VUP9YmDcYa+eyq4ca7iLqWA=="],

    "electron-to-chromium": ["electron-to-chromium@1.5.192", "", {}, "sha512-rP8Ez0w7UNw/9j5eSXCe10o1g/8B1P5SM90PCCMVkIRQn2R0LEHWz4Eh9RnxkniuDe1W0cTSOB3MLlkTGDcuCg=="],

    "embla-carousel": ["embla-carousel@8.6.0", "", {}, "sha512-SjWyZBHJPbqxHOzckOfo8lHisEaJWmwd23XppYFYVh10bU66/Pn5tkVkbkCMZVdbUE5eTCI2nD8OyIP4Z+uwkA=="],

    "embla-carousel-react": ["embla-carousel-react@8.6.0", "", { "dependencies": { "embla-carousel": "8.6.0", "embla-carousel-reactive-utils": "8.6.0" }, "peerDependencies": { "react": "^16.8.0 || ^17.0.1 || ^18.0.0 || ^19.0.0 || ^19.0.0-rc" } }, "sha512-0/PjqU7geVmo6F734pmPqpyHqiM99olvyecY7zdweCw+6tKEXnrE90pBiBbMMU8s5tICemzpQ3hi5EpxzGW+JA=="],

    "embla-carousel-reactive-utils": ["embla-carousel-reactive-utils@8.6.0", "", { "peerDependencies": { "embla-carousel": "8.6.0" } }, "sha512-fMVUDUEx0/uIEDM0Mz3dHznDhfX+znCCDCeIophYb1QGVM7YThSWX+wz11zlYwWFOr74b4QLGg0hrGPJeG2s4A=="],

    "emoji-regex": ["emoji-regex@9.2.2", "", {}, "sha512-L18DaJsXSUk2+42pv8mLs5jJT2hqFkFE4j21wOmgbUqsZ2hL72NsUU785g9RXgo3s0ZNgVl42TiHp3ZtOv/Vyg=="],

    "entities": ["entities@6.0.1", "", {}, "sha512-aN97NXWF6AWBTahfVOIrB/NShkzi5H7F9r1s9mD3cDj4Ko5f2qhhVoYMibXF7GlLveb/D2ioWay8lxI97Ven3g=="],

    "es-define-property": ["es-define-property@1.0.1", "", {}, "sha512-e3nRfgfUZ4rNGL232gUgX06QNyyez04KdjFrF+LTRoOXmrOgFKDg4BCdsjW8EnT69eqdYGmRpJwiPVYNrCaW3g=="],

    "es-errors": ["es-errors@1.3.0", "", {}, "sha512-Zf5H2Kxt2xjTvbJvP2ZWLEICxA6j+hAmMzIlypy4xcBg1vKVnx89Wy0GbS+kf5cwCVFFzdCFh2XSCFNULS6csw=="],

    "es-module-lexer": ["es-module-lexer@1.7.0", "", {}, "sha512-jEQoCwk8hyb2AZziIOLhDqpm5+2ww5uIE6lkO/6jcOCusfk6LhMHpXXfBLXTZ7Ydyt0j4VoUQv6uGNYbdW+kBA=="],

    "es-object-atoms": ["es-object-atoms@1.1.1", "", { "dependencies": { "es-errors": "^1.3.0" } }, "sha512-FGgH2h8zKNim9ljj7dankFPcICIK9Cp5bm+c2gQSYePhpaG5+esrLODihIorn+Pe6FGJzWhXQotPv73jTaldXA=="],

    "es-set-tostringtag": ["es-set-tostringtag@2.1.0", "", { "dependencies": { "es-errors": "^1.3.0", "get-intrinsic": "^1.2.6", "has-tostringtag": "^1.0.2", "hasown": "^2.0.2" } }, "sha512-j6vWzfrGVfyXxge+O0x5sh6cvxAog0a/4Rdd2K36zCMV5eJ+/+tOAngRO8cODMNWbVRdVlmGZQL2YS3yR8bIUA=="],

    "esbuild": ["esbuild@0.25.0", "", { "optionalDependencies": { "@esbuild/aix-ppc64": "0.25.0", "@esbuild/android-arm": "0.25.0", "@esbuild/android-arm64": "0.25.0", "@esbuild/android-x64": "0.25.0", "@esbuild/darwin-arm64": "0.25.0", "@esbuild/darwin-x64": "0.25.0", "@esbuild/freebsd-arm64": "0.25.0", "@esbuild/freebsd-x64": "0.25.0", "@esbuild/linux-arm": "0.25.0", "@esbuild/linux-arm64": "0.25.0", "@esbuild/linux-ia32": "0.25.0", "@esbuild/linux-loong64": "0.25.0", "@esbuild/linux-mips64el": "0.25.0", "@esbuild/linux-ppc64": "0.25.0", "@esbuild/linux-riscv64": "0.25.0", "@esbuild/linux-s390x": "0.25.0", "@esbuild/linux-x64": "0.25.0", "@esbuild/netbsd-arm64": "0.25.0", "@esbuild/netbsd-x64": "0.25.0", "@esbuild/openbsd-arm64": "0.25.0", "@esbuild/openbsd-x64": "0.25.0", "@esbuild/sunos-x64": "0.25.0", "@esbuild/win32-arm64": "0.25.0", "@esbuild/win32-ia32": "0.25.0", "@esbuild/win32-x64": "0.25.0" }, "bin": "bin/esbuild" }, "sha512-BXq5mqc8ltbaN34cDqWuYKyNhX8D/Z0J1xdtdQ8UcIIIyJyz+ZMKUt58tF3SrZ85jcfN/PZYhjR5uDQAYNVbuw=="],

    "escalade": ["escalade@3.2.0", "", {}, "sha512-WUj2qlxaQtO4g6Pq5c29GTcWGDyd8itL8zTlipgECz3JesAiiOKotd8JU6otB3PACgG6xkJUyVhboMS+bje/jA=="],

    "escape-string-regexp": ["escape-string-regexp@4.0.0", "", {}, "sha512-TtpcNJ3XAzx3Gq8sWRzJaVajRs0uVxA2YAkdb1jm2YkPz4G6egUFAyA3n5vtEIZefPk5Wa4UXbKuS5fKkJWdgA=="],

    "escodegen": ["escodegen@2.1.0", "", { "dependencies": { "esprima": "^4.0.1", "estraverse": "^5.2.0", "esutils": "^2.0.2" }, "optionalDependencies": { "source-map": "~0.6.1" }, "bin": { "escodegen": "bin/escodegen.js", "esgenerate": "bin/esgenerate.js" } }, "sha512-2NlIDTwUWJN0mRPQOdtQBzbUHvdGY2P1VXSyU83Q3xKxM7WHX2Ql8dKq782Q9TgQUNOLEzEYu9bzLNj1q88I5w=="],

    "eslint": ["eslint@9.32.0", "", { "dependencies": { "@eslint-community/eslint-utils": "^4.2.0", "@eslint-community/regexpp": "^4.12.1", "@eslint/config-array": "^0.21.0", "@eslint/config-helpers": "^0.3.0", "@eslint/core": "^0.15.0", "@eslint/eslintrc": "^3.3.1", "@eslint/js": "9.32.0", "@eslint/plugin-kit": "^0.3.4", "@humanfs/node": "^0.16.6", "@humanwhocodes/module-importer": "^1.0.1", "@humanwhocodes/retry": "^0.4.2", "@types/estree": "^1.0.6", "@types/json-schema": "^7.0.15", "ajv": "^6.12.4", "chalk": "^4.0.0", "cross-spawn": "^7.0.6", "debug": "^4.3.2", "escape-string-regexp": "^4.0.0", "eslint-scope": "^8.4.0", "eslint-visitor-keys": "^4.2.1", "espree": "^10.4.0", "esquery": "^1.5.0", "esutils": "^2.0.2", "fast-deep-equal": "^3.1.3", "file-entry-cache": "^8.0.0", "find-up": "^5.0.0", "glob-parent": "^6.0.2", "ignore": "^5.2.0", "imurmurhash": "^0.1.4", "is-glob": "^4.0.0", "json-stable-stringify-without-jsonify": "^1.0.1", "lodash.merge": "^4.6.2", "minimatch": "^3.1.2", "natural-compare": "^1.4.0", "optionator": "^0.9.3" }, "peerDependencies": { "jiti": "*" }, "bin": "bin/eslint.js" }, "sha512-LSehfdpgMeWcTZkWZVIJl+tkZ2nuSkyyB9C27MZqFWXuph7DvaowgcTvKqxvpLW1JZIk8PN7hFY3Rj9LQ7m7lg=="],

    "eslint-plugin-react-hooks": ["eslint-plugin-react-hooks@5.2.0", "", { "peerDependencies": { "eslint": "^3.0.0 || ^4.0.0 || ^5.0.0 || ^6.0.0 || ^7.0.0 || ^8.0.0-0 || ^9.0.0" } }, "sha512-+f15FfK64YQwZdJNELETdn5ibXEUQmW1DZL6KXhNnc2heoy/sg9VJJeT7n8TlMWouzWqSWavFkIhHyIbIAEapg=="],

    "eslint-plugin-react-refresh": ["eslint-plugin-react-refresh@0.4.20", "", { "peerDependencies": { "eslint": ">=8.40" } }, "sha512-XpbHQ2q5gUF8BGOX4dHe+71qoirYMhApEPZ7sfhF/dNnOF1UXnCMGZf79SFTBO7Bz5YEIT4TMieSlJBWhP9WBA=="],

    "eslint-scope": ["eslint-scope@8.4.0", "", { "dependencies": { "esrecurse": "^4.3.0", "estraverse": "^5.2.0" } }, "sha512-sNXOfKCn74rt8RICKMvJS7XKV/Xk9kA7DyJr8mJik3S7Cwgy3qlkkmyS2uQB3jiJg6VNdZd/pDBJu0nvG2NlTg=="],

    "eslint-visitor-keys": ["eslint-visitor-keys@4.2.1", "", {}, "sha512-Uhdk5sfqcee/9H/rCOJikYz67o0a2Tw2hGRPOG2Y1R2dg7brRe1uG0yaNQDHu+TO/uQPF/5eCapvYSmHUjt7JQ=="],

    "espree": ["espree@10.4.0", "", { "dependencies": { "acorn": "^8.15.0", "acorn-jsx": "^5.3.2", "eslint-visitor-keys": "^4.2.1" } }, "sha512-j6PAQ2uUr79PZhBjP5C5fhl8e39FmRnOjsD5lGnWrFU8i2G776tBK7+nP8KuQUTTyAZUwfQqXAgrVH5MbH9CYQ=="],

    "esprima": ["esprima@4.0.1", "", { "bin": { "esparse": "bin/esparse.js", "esvalidate": "bin/esvalidate.js" } }, "sha512-eGuFFw7Upda+g4p+QHvnW0RyTX/SVeJBDM/gCtMARO0cLuT2HcEKnTPvhjV6aGeqrCB/sbNop0Kszm0jsaWU4A=="],

    "esquery": ["esquery@1.6.0", "", { "dependencies": { "estraverse": "^5.1.0" } }, "sha512-ca9pw9fomFcKPvFLXhBKUK90ZvGibiGOvRJNbjljY7s7uq/5YO4BOzcYtJqExdx99rF6aAcnRxHmcUHcz6sQsg=="],

    "esrecurse": ["esrecurse@4.3.0", "", { "dependencies": { "estraverse": "^5.2.0" } }, "sha512-KmfKL3b6G+RXvP8N1vr3Tq1kL/oCFgn2NYXEtqP8/L3pKapUA4G8cFVaoF3SU323CD4XypR/ffioHmkti6/Tag=="],

    "estraverse": ["estraverse@5.3.0", "", {}, "sha512-MMdARuVEQziNTeJD8DgMqmhwR11BRQ/cBP+pLtYdSTnf3MIO8fFeiINEbX36ZdNlfU/7A9f3gUw49B3oQsvwBA=="],

    "estree-util-is-identifier-name": ["estree-util-is-identifier-name@3.0.0", "", {}, "sha512-hFtqIDZTIUZ9BXLb8y4pYGyk6+wekIivNVTcmvk8NoOh+VeRn5y6cEHzbURrWbfp1fIqdVipilzj+lfaadNZmg=="],

    "estree-walker": ["estree-walker@3.0.3", "", { "dependencies": { "@types/estree": "^1.0.0" } }, "sha512-7RUKfXgSMMkzt6ZuXmqapOurLGPPfgj6l9uRZ7lRGolvk0y2yocc35LdcxKC5PQZdn2DMqioAQ2NoWcrTKmm6g=="],

    "esutils": ["esutils@2.0.3", "", {}, "sha512-kVscqXk4OCp68SZ0dkgEKVi6/8ij300KBWTJq32P/dYeWTSwK41WyTxalN1eRmA5Z9UU/LX9D7FWSmV9SAYx6g=="],

    "eventemitter3": ["eventemitter3@4.0.7", "", {}, "sha512-8guHBZCwKnFhYdHr2ysuRWErTwhoN2X8XELRlrRwpmfeY2jjuUN4taQMsULKUVo1K4DvZl+0pgfyoysHxvmvEw=="],

    "expect-type": ["expect-type@1.3.0", "", {}, "sha512-knvyeauYhqjOYvQ66MznSMs83wmHrCycNEN6Ao+2AeYEfxUIkuiVxdEa1qlGEPK+We3n0THiDciYSsCcgW/DoA=="],

    "extend": ["extend@3.0.2", "", {}, "sha512-fjquC59cD7CyW6urNXK0FBufkZcoiGG80wTuPujX590cB5Ttln20E2UB4S/WARVqhXffZl2LNgS+gQdPIIim/g=="],

    "fast-deep-equal": ["fast-deep-equal@3.1.3", "", {}, "sha512-f3qQ9oQy9j2AhBe/H9VC91wLmKBCCU/gDOnKNAYG5hswO7BLKj09Hc5HYNz9cGI++xlpDCIgDaitVs03ATR84Q=="],

    "fast-equals": ["fast-equals@5.2.2", "", {}, "sha512-V7/RktU11J3I36Nwq2JnZEM7tNm17eBJz+u25qdxBZeCKiX6BkVSZQjwWIr+IobgnZy+ag73tTZgZi7tr0LrBw=="],

    "fast-glob": ["fast-glob@3.3.2", "", { "dependencies": { "@nodelib/fs.stat": "^2.0.2", "@nodelib/fs.walk": "^1.2.3", "glob-parent": "^5.1.2", "merge2": "^1.3.0", "micromatch": "^4.0.4" } }, "sha512-oX2ruAFQwf/Orj8m737Y5adxDQO0LAB7/S5MnxCdTNDd4p6BsyIVsv9JQsATbTSq8KHRpLwIHbVlUNatxd+1Ow=="],

    "fast-json-stable-stringify": ["fast-json-stable-stringify@2.1.0", "", {}, "sha512-lhd/wF+Lk98HZoTCtlVraHtfh5XYijIjalXck7saUtuanSDyLMxnHhSXEDJqHxD7msR8D0uCmqlkwjCV8xvwHw=="],

    "fast-levenshtein": ["fast-levenshtein@2.0.6", "", {}, "sha512-DCXu6Ifhqcks7TZKY3Hxp3y6qphY5SJZmrWMDrKcERSOXWQdMhU9Ig/PYrzyw/ul9jOIyh0N4M0tbC5hodg8dw=="],

    "fastq": ["fastq@1.17.1", "", { "dependencies": { "reusify": "^1.0.4" } }, "sha512-sRVD3lWVIXWg6By68ZN7vho9a1pQcN/WBFaAAsDDFzlJjvoGx0P8z7V1t72grFJfJhu3YPZBuu25f7Kaw2jN1w=="],

    "fdir": ["fdir@6.5.0", "", { "peerDependencies": { "picomatch": "^3 || ^4" } }, "sha512-tIbYtZbucOs0BRGqPJkshJUYdL+SDH7dVM8gjy+ERp3WAUjLEFJE+02kanyHtwjWOnwrKYBiwAmM0p4kLJAnXg=="],

    "file-entry-cache": ["file-entry-cache@8.0.0", "", { "dependencies": { "flat-cache": "^4.0.0" } }, "sha512-XXTUwCvisa5oacNGRP9SfNtYBNAMi+RPwBFmblZEF7N7swHYQS6/Zfk7SRwx4D5j3CH211YNRco1DEMNVfZCnQ=="],

    "fill-range": ["fill-range@7.1.1", "", { "dependencies": { "to-regex-range": "^5.0.1" } }, "sha512-YsGpe3WHLK8ZYi4tWDg2Jy3ebRz2rXowDxnld4bkQB00cc/1Zw9AWnC0i9ztDJitivtQvaI9KaLyKrc+hBW0yg=="],

    "find-up": ["find-up@5.0.0", "", { "dependencies": { "locate-path": "^6.0.0", "path-exists": "^4.0.0" } }, "sha512-78/PXT1wlLLDgTzDs7sjq9hzz0vXD+zn+7wypEe4fXQxCmdmqfGsEPQxmiCSQI3ajFV91bVSsvNtrJRiW6nGng=="],

    "flat-cache": ["flat-cache@4.0.1", "", { "dependencies": { "flatted": "^3.2.9", "keyv": "^4.5.4" } }, "sha512-f7ccFPK3SXFHpx15UIGyRJ/FJQctuKZ0zVuN3frBo4HnK3cay9VEW0R6yPYFHC0AgqhukPzKjq22t5DmAyqGyw=="],

    "flatted": ["flatted@3.3.1", "", {}, "sha512-X8cqMLLie7KsNUDSdzeN8FYK9rEt4Dt67OsG/DNGnYTSDBG4uFAJFBnUeiV+zCVAvwFy56IjM9sH51jVaEhNxw=="],

    "foreground-child": ["foreground-child@3.3.0", "", { "dependencies": { "cross-spawn": "^7.0.0", "signal-exit": "^4.0.1" } }, "sha512-Ld2g8rrAyMYFXBhEqMz8ZAHBi4J4uS1i/CxGMDnjyFWddMXLVcDp051DZfu+t7+ab7Wv6SMqpWmyFIj5UbfFvg=="],

    "form-data": ["form-data@4.0.5", "", { "dependencies": { "asynckit": "^0.4.0", "combined-stream": "^1.0.8", "es-set-tostringtag": "^2.1.0", "hasown": "^2.0.2", "mime-types": "^2.1.12" } }, "sha512-8RipRLol37bNs2bhoV67fiTEvdTrbMUYcFTiy3+wuuOnUog2QBHCZWXDRijWQfAkhBj2Uf5UnVaiWwA5vdd82w=="],

    "fraction.js": ["fraction.js@4.3.7", "", {}, "sha512-ZsDfxO51wGAXREY55a7la9LScWpwv9RxIrYABrlvOFBlH/ShPnrtsXeuUIfXKKOVicNxQ+o8JTbJvjS4M89yew=="],

    "framer-motion": ["framer-motion@12.35.0", "", { "dependencies": { "motion-dom": "^12.35.0", "motion-utils": "^12.29.2", "tslib": "^2.4.0" }, "peerDependencies": { "@emotion/is-prop-valid": "*", "react": "^18.0.0 || ^19.0.0", "react-dom": "^18.0.0 || ^19.0.0" }, "optionalPeers": ["@emotion/is-prop-valid"] }, "sha512-w8hghCMQ4oq10j6aZh3U2yeEQv5K69O/seDI/41PK4HtgkLrcBovUNc0ayBC3UyyU7V1mrY2yLzvYdWJX9pGZQ=="],

    "fsevents": ["fsevents@2.3.3", "", { "os": "darwin" }, "sha512-5xoDfX+fL7faATnagmWPpbFtwh/R77WmMMqqHGS65C3vvB0YHrgF+B1YmZ3441tMj5n63k0212XNoJwzlhffQw=="],

    "function-bind": ["function-bind@1.1.2", "", {}, "sha512-7XHNxH7qX9xG5mIwxkhumTox/MIRNcOgDrxWsMt2pAr23WHp6MrRlN7FBSFpCpr+oVO0F744iUgR82nJMfG2SA=="],

    "get-intrinsic": ["get-intrinsic@1.3.0", "", { "dependencies": { "call-bind-apply-helpers": "^1.0.2", "es-define-property": "^1.0.1", "es-errors": "^1.3.0", "es-object-atoms": "^1.1.1", "function-bind": "^1.1.2", "get-proto": "^1.0.1", "gopd": "^1.2.0", "has-symbols": "^1.1.0", "hasown": "^2.0.2", "math-intrinsics": "^1.1.0" } }, "sha512-9fSjSaos/fRIVIp+xSJlE6lfwhES7LNtKaCBIamHsjr2na1BiABJPo0mOjjz8GJDURarmCPGqaiVg5mfjb98CQ=="],

    "get-nonce": ["get-nonce@1.0.1", "", {}, "sha512-FJhYRoDaiatfEkUK8HKlicmu/3SGFD51q3itKDGoSTysQJBnfOcxU5GxnhE1E6soB76MbT0MBtnKJuXyAx+96Q=="],

    "get-proto": ["get-proto@1.0.1", "", { "dependencies": { "dunder-proto": "^1.0.1", "es-object-atoms": "^1.0.0" } }, "sha512-sTSfBjoXBp89JvIKIefqw7U2CCebsc74kiY6awiGogKtoSGbgjYE/G/+l9sF3MWFPNc9IcoOC4ODfKHfxFmp0g=="],

    "glob": ["glob@10.4.5", "", { "dependencies": { "foreground-child": "^3.1.0", "jackspeak": "^3.1.2", "minimatch": "^9.0.4", "minipass": "^7.1.2", "package-json-from-dist": "^1.0.0", "path-scurry": "^1.11.1" }, "bin": "dist/esm/bin.mjs" }, "sha512-7Bv8RF0k6xjo7d4A/PxYLbUCfb6c+Vpd2/mB2yRDlew7Jb5hEXiCD9ibfO7wpk8i4sevK6DFny9h7EYbM3/sHg=="],

    "glob-parent": ["glob-parent@6.0.2", "", { "dependencies": { "is-glob": "^4.0.3" } }, "sha512-XxwI8EOhVQgWp6iDL+3b0r86f4d6AX6zSU55HfB4ydCEuXLXc5FcYeOu+nnGftS4TEju/11rt4KJPTMgbfmv4A=="],

    "globals": ["globals@15.15.0", "", {}, "sha512-7ACyT3wmyp3I61S4fG682L0VA2RGD9otkqGJIwNUMF1SWUombIIk+af1unuDYgMm082aHYwD+mzJvv9Iu8dsgg=="],

    "gopd": ["gopd@1.2.0", "", {}, "sha512-ZUKRh6/kUFoAiTAtTYPZJ3hw9wNxx+BIBOijnlG9PnrJsCcSjs1wyyD6vJpaYtgnzDrKYRSqf3OO6Rfa93xsRg=="],

    "graphemer": ["graphemer@1.4.0", "", {}, "sha512-EtKwoO6kxCL9WO5xipiHTZlSzBm7WLT627TqC/uVRd0HKmq8NXyebnNYxDoBi7wt8eTWrUrKXCOVaFq9x1kgag=="],

    "has-flag": ["has-flag@4.0.0", "", {}, "sha512-EykJT/Q1KjTWctppgIAgfSO0tKVuZUjhgMr17kqTumMl6Afv3EISleU7qZUzoXDFTAHTDC4NOoG/ZxU3EvlMPQ=="],

    "has-symbols": ["has-symbols@1.1.0", "", {}, "sha512-1cDNdwJ2Jaohmb3sg4OmKaMBwuC48sYni5HUw2DvsC8LjGTLK9h+eb1X6RyuOHe4hT0ULCW68iomhjUoKUqlPQ=="],

    "has-tostringtag": ["has-tostringtag@1.0.2", "", { "dependencies": { "has-symbols": "^1.0.3" } }, "sha512-NqADB8VjPFLM2V0VvHUewwwsw0ZWBaIdgo+ieHtK3hasLz4qeCRjYcqfB6AQrBggRKppKF8L52/VqdVsO47Dlw=="],

    "hasown": ["hasown@2.0.2", "", { "dependencies": { "function-bind": "^1.1.2" } }, "sha512-0hJU9SCPvmMzIBdZFqNPXWa6dqh7WdH0cII9y+CyS8rG3nL48Bclra9HmKhVVUHyPWNH5Y7xDwAB7bfgSjkUMQ=="],

    "hast-util-to-jsx-runtime": ["hast-util-to-jsx-runtime@2.3.6", "", { "dependencies": { "@types/estree": "^1.0.0", "@types/hast": "^3.0.0", "@types/unist": "^3.0.0", "comma-separated-tokens": "^2.0.0", "devlop": "^1.0.0", "estree-util-is-identifier-name": "^3.0.0", "hast-util-whitespace": "^3.0.0", "mdast-util-mdx-expression": "^2.0.0", "mdast-util-mdx-jsx": "^3.0.0", "mdast-util-mdxjs-esm": "^2.0.0", "property-information": "^7.0.0", "space-separated-tokens": "^2.0.0", "style-to-js": "^1.0.0", "unist-util-position": "^5.0.0", "vfile-message": "^4.0.0" } }, "sha512-zl6s8LwNyo1P9uw+XJGvZtdFF1GdAkOg8ujOw+4Pyb76874fLps4ueHXDhXWdk6YHQ6OgUtinliG7RsYvCbbBg=="],

    "hast-util-whitespace": ["hast-util-whitespace@3.0.0", "", { "dependencies": { "@types/hast": "^3.0.0" } }, "sha512-88JUN06ipLwsnv+dVn+OIYOvAuvBMy/Qoi6O7mQHxdPXpjy+Cd6xRkWwux7DKO+4sYILtLBRIKgsdpS2gQc7qw=="],

    "html-encoding-sniffer": ["html-encoding-sniffer@3.0.0", "", { "dependencies": { "whatwg-encoding": "^2.0.0" } }, "sha512-oWv4T4yJ52iKrufjnyZPkrN0CH3QnrUqdB6In1g5Fe1mia8GmF36gnfNySxoZtxD5+NmYw1EElVXiBk93UeskA=="],

    "html-url-attributes": ["html-url-attributes@3.0.1", "", {}, "sha512-ol6UPyBWqsrO6EJySPz2O7ZSr856WDrEzM5zMqp+FJJLGMW35cLYmmZnl0vztAZxRUoNZJFTCohfjuIJ8I4QBQ=="],

    "http-proxy-agent": ["http-proxy-agent@5.0.0", "", { "dependencies": { "@tootallnate/once": "2", "agent-base": "6", "debug": "4" } }, "sha512-n2hY8YdoRE1i7r6M0w9DIw5GgZN0G25P8zLCRQ8rjXtTU3vsNFBI/vWK/UIeE6g5MUUz6avwAPXmL6Fy9D/90w=="],

    "https-proxy-agent": ["https-proxy-agent@5.0.1", "", { "dependencies": { "agent-base": "6", "debug": "4" } }, "sha512-dFcAjpTQFgoLMzC2VwU+C/CbS7uRL0lWmxDITmqm7C+7F0Odmj6s9l6alZc6AELXhrnggM2CeWSXHGOdX2YtwA=="],

    "iceberg-js": ["iceberg-js@0.8.1", "", {}, "sha512-1dhVQZXhcHje7798IVM+xoo/1ZdVfzOMIc8/rgVSijRK38EDqOJoGula9N/8ZI5RD8QTxNQtK/Gozpr+qUqRRA=="],

    "iconv-lite": ["iconv-lite@0.6.3", "", { "dependencies": { "safer-buffer": ">= 2.1.2 < 3.0.0" } }, "sha512-4fCk79wshMdzMp2rH06qWrJE4iolqLhCUH+OiuIgU++RB0+94NlDL81atO7GX55uUKueo0txHNtvEyI6D7WdMw=="],

    "ignore": ["ignore@5.3.2", "", {}, "sha512-hsBTNUqQTDwkWtcdYI2i06Y/nUBEsNEDJKjWdigLvegy8kDuJAS8uRlpkkcQpyEXL0Z/pjDy5HBmMjRCJ2gq+g=="],

    "import-fresh": ["import-fresh@3.3.1", "", { "dependencies": { "parent-module": "^1.0.0", "resolve-from": "^4.0.0" } }, "sha512-TR3KfrTZTYLPB6jUjfx6MF9WcWrHL9su5TObK4ZkYgBdWKPOFoSoQIdEuTuR82pmtxH2spWG9h6etwfr1pLBqQ=="],

    "imurmurhash": ["imurmurhash@0.1.4", "", {}, "sha512-JmXMZ6wuvDmLiHEml9ykzqO6lwFbof0GG4IkcGaENdCRDDmMVnny7s5HsIgHCbaq0w2MyPhDqkhTUgS2LU2PHA=="],

    "indent-string": ["indent-string@4.0.0", "", {}, "sha512-EdDDZu4A2OyIK7Lr/2zG+w5jmbuk1DVBnEwREQvBzspBJkCEbRa8GxU1lghYcaGJCnRWibjDXlq779X1/y5xwg=="],

    "inline-style-parser": ["inline-style-parser@0.2.7", "", {}, "sha512-Nb2ctOyNR8DqQoR0OwRG95uNWIC0C1lCgf5Naz5H6Ji72KZ8OcFZLz2P5sNgwlyoJ8Yif11oMuYs5pBQa86csA=="],

    "input-otp": ["input-otp@1.4.2", "", { "peerDependencies": { "react": "^16.8 || ^17.0 || ^18.0 || ^19.0.0 || ^19.0.0-rc", "react-dom": "^16.8 || ^17.0 || ^18.0 || ^19.0.0 || ^19.0.0-rc" } }, "sha512-l3jWwYNvrEa6NTCt7BECfCm48GvwuZzkoeG3gBL2w4CHeOXW3eKFmf9UNYkNfYc3mxMrthMnxjIE07MT0zLBQA=="],

    "internmap": ["internmap@2.0.3", "", {}, "sha512-5Hh7Y1wQbvY5ooGgPbDaL5iYLAPzMTUrjMulskHLH6wnv/A+1q5rgEaiuqEjB+oxGXIVZs1FF+R/KPN3ZSQYYg=="],

    "is-alphabetical": ["is-alphabetical@2.0.1", "", {}, "sha512-FWyyY60MeTNyeSRpkM2Iry0G9hpr7/9kD40mD/cGQEuilcZYS4okz8SN2Q6rLCJ8gbCt6fN+rC+6tMGS99LaxQ=="],

    "is-alphanumerical": ["is-alphanumerical@2.0.1", "", { "dependencies": { "is-alphabetical": "^2.0.0", "is-decimal": "^2.0.0" } }, "sha512-hmbYhX/9MUMF5uh7tOXyK/n0ZvWpad5caBA17GsC6vyuCqaWliRG5K1qS9inmUhEMaOBIW7/whAnSwveW/LtZw=="],

    "is-binary-path": ["is-binary-path@2.1.0", "", { "dependencies": { "binary-extensions": "^2.0.0" } }, "sha512-ZMERYes6pDydyuGidse7OsHxtbI7WVeUEozgR/g7rd0xUimYNlvZRE/K2MgZTjWy725IfelLeVcEM97mmtRGXw=="],

    "is-core-module": ["is-core-module@2.15.1", "", { "dependencies": { "hasown": "^2.0.2" } }, "sha512-z0vtXSwucUJtANQWldhbtbt7BnL0vxiFjIdDLAatwhDYty2bad6s+rijD6Ri4YuYJubLzIJLUidCh09e1djEVQ=="],

    "is-decimal": ["is-decimal@2.0.1", "", {}, "sha512-AAB9hiomQs5DXWcRB1rqsxGUstbRroFOPPVAomNk/3XHR5JyEZChOyTWe2oayKnsSsr/kcGqF+z6yuH6HHpN0A=="],

    "is-extglob": ["is-extglob@2.1.1", "", {}, "sha512-SbKbANkN603Vi4jEZv49LeVJMn4yGwsbzZworEoyEiutsN3nJYdbO36zfhGJ6QEDpOZIFkDtnq5JRxmvl3jsoQ=="],

    "is-fullwidth-code-point": ["is-fullwidth-code-point@3.0.0", "", {}, "sha512-zymm5+u+sCsSWyD9qNaejV3DFvhCKclKdizYaJUuHA83RLjb7nSuGnddCHGv0hk+KY7BMAlsWeK4Ueg6EV6XQg=="],

    "is-glob": ["is-glob@4.0.3", "", { "dependencies": { "is-extglob": "^2.1.1" } }, "sha512-xelSayHH36ZgE7ZWhli7pW34hNbNl8Ojv5KVmkJD4hBdD3th8Tfk9vYasLM+mXWOZhFkgZfxhLSnrwRr4elSSg=="],

    "is-hexadecimal": ["is-hexadecimal@2.0.1", "", {}, "sha512-DgZQp241c8oO6cA1SbTEWiXeoxV42vlcJxgH+B3hi1AiqqKruZR3ZGF8In3fj4+/y/7rHvlOZLZtgJ/4ttYGZg=="],

    "is-number": ["is-number@7.0.0", "", {}, "sha512-41Cifkg6e8TylSpdtTpeLVMqvSBEVzTttHvERD741+pnZ8ANv0004MRL43QKPDlK9cGvNp6NZWZUBlbGXYxxng=="],

    "is-plain-obj": ["is-plain-obj@4.1.0", "", {}, "sha512-+Pgi+vMuUNkJyExiMBt5IlFoMyKnr5zhJ4Uspz58WOhBF5QoIZkFyNHIbBAtHwzVAgk5RtndVNsDRN61/mmDqg=="],

    "is-potential-custom-element-name": ["is-potential-custom-element-name@1.0.1", "", {}, "sha512-bCYeRA2rVibKZd+s2625gGnGF/t7DSqDs4dP7CrLA1m7jKWz6pps0LpYLJN8Q64HtmPKJ1hrN3nzPNKFEKOUiQ=="],

    "isexe": ["isexe@2.0.0", "", {}, "sha512-RHxMLp9lnKHGHRng9QFhRCMbYAcVpn69smSGcq3f36xjgVVWThj4qqLbTLlq7Ssj8B+fIQ1EuCEGI2lKsyQeIw=="],

    "jackspeak": ["jackspeak@3.4.3", "", { "dependencies": { "@isaacs/cliui": "^8.0.2" }, "optionalDependencies": { "@pkgjs/parseargs": "^0.11.0" } }, "sha512-OGlZQpz2yfahA/Rd1Y8Cd9SIEsqvXkLVoSw/cgwhnhFMDbsQFeZYoJJ7bIZBS9BcamUW96asq/npPWugM+RQBw=="],

    "jiti": ["jiti@1.21.6", "", { "bin": "bin/jiti.js" }, "sha512-2yTgeWTWzMWkHu6Jp9NKgePDaYHbntiwvYuuJLbbN9vl7DC9DvXKOB2BC3ZZ92D3cvV/aflH0osDfwpHepQ53w=="],

    "js-tokens": ["js-tokens@4.0.0", "", {}, "sha512-RdJUflcE3cUzKiMqQgsCu06FPu9UdIJO0beYbPhHN4k6apgJtifcoCtT9bcxOpYBtpD2kCM6Sbzg4CausW/PKQ=="],

    "js-yaml": ["js-yaml@4.1.0", "", { "dependencies": { "argparse": "^2.0.1" }, "bin": "bin/js-yaml.js" }, "sha512-wpxZs9NoxZaJESJGIZTyDEaYpl0FKSA+FB9aJiyemKhMwkxQg63h4T1KJgUGHpTqPDNRcmmYLugrRjJlBtWvRA=="],

    "jsdom": ["jsdom@20.0.3", "", { "dependencies": { "abab": "^2.0.6", "acorn": "^8.8.1", "acorn-globals": "^7.0.0", "cssom": "^0.5.0", "cssstyle": "^2.3.0", "data-urls": "^3.0.2", "decimal.js": "^10.4.2", "domexception": "^4.0.0", "escodegen": "^2.0.0", "form-data": "^4.0.0", "html-encoding-sniffer": "^3.0.0", "http-proxy-agent": "^5.0.0", "https-proxy-agent": "^5.0.1", "is-potential-custom-element-name": "^1.0.1", "nwsapi": "^2.2.2", "parse5": "^7.1.1", "saxes": "^6.0.0", "symbol-tree": "^3.2.4", "tough-cookie": "^4.1.2", "w3c-xmlserializer": "^4.0.0", "webidl-conversions": "^7.0.0", "whatwg-encoding": "^2.0.0", "whatwg-mimetype": "^3.0.0", "whatwg-url": "^11.0.0", "ws": "^8.11.0", "xml-name-validator": "^4.0.0" }, "peerDependencies": { "canvas": "^2.5.0" }, "optionalPeers": ["canvas"] }, "sha512-SYhBvTh89tTfCD/CRdSOm13mOBa42iTaTyfyEWBdKcGdPxPtLFBXuHR8XHb33YNYaP+lLbmSvBTsnoesCNJEsQ=="],

    "json-buffer": ["json-buffer@3.0.1", "", {}, "sha512-4bV5BfR2mqfQTJm+V5tPPdf+ZpuhiIvTuAB5g8kcrXOZpTT/QwwVRWBywX1ozr6lEuPdbHxwaJlm9G6mI2sfSQ=="],

    "json-schema-traverse": ["json-schema-traverse@0.4.1", "", {}, "sha512-xbbCH5dCYU5T8LcEhhuh7HJ88HXuW3qsI3Y0zOZFKfZEHcpWiHU/Jxzk629Brsab/mMiHQti9wMP+845RPe3Vg=="],

    "json-stable-stringify-without-jsonify": ["json-stable-stringify-without-jsonify@1.0.1", "", {}, "sha512-Bdboy+l7tA3OGW6FjyFHWkP5LuByj1Tk33Ljyq0axyzdk9//JSi2u3fP1QSmd1KNwq6VOKYGlAu87CisVir6Pw=="],

    "keyv": ["keyv@4.5.4", "", { "dependencies": { "json-buffer": "3.0.1" } }, "sha512-oxVHkHR/EJf2CNXnWxRLW6mg7JyCCUcG0DtEGmL2ctUo1PNTin1PUil+r/+4r5MpVgC/fn1kjsx7mjSujKqIpw=="],

    "levn": ["levn@0.4.1", "", { "dependencies": { "prelude-ls": "^1.2.1", "type-check": "~0.4.0" } }, "sha512-+bT2uH4E5LGE7h/n3evcS/sQlJXCpIp6ym8OWJ5eV6+67Dsql/LaaT7qJBAt2rzfoa/5QBGBhxDix1dMt2kQKQ=="],

    "lilconfig": ["lilconfig@3.1.3", "", {}, "sha512-/vlFKAoH5Cgt3Ie+JLhRbwOsCQePABiU3tJ1egGvyQ+33R/vcwM2Zl2QR/LzjsBeItPt3oSVXapn+m4nQDvpzw=="],

    "lines-and-columns": ["lines-and-columns@1.2.4", "", {}, "sha512-7ylylesZQ/PV29jhEDl3Ufjo6ZX7gCqJr5F7PKrqc93v7fzSymt1BpwEU8nAUXs8qzzvqhbjhK5QZg6Mt/HkBg=="],

    "locate-path": ["locate-path@6.0.0", "", { "dependencies": { "p-locate": "^5.0.0" } }, "sha512-iPZK6eYjbxRu3uB4/WZ3EsEIMJFMqAoopl3R+zuq0UjcAm/MO6KCweDgPfP3elTztoKP3KtnVHxTn2NHBSDVUw=="],

    "lodash": ["lodash@4.17.21", "", {}, "sha512-v2kDEe57lecTulaDIuNTPy3Ry4gLGJ6Z1O3vE1krgXZNrsQ+LFTGHVxVjcXPs17LhbZVGedAJv8XZ1tvj5FvSg=="],

    "lodash.castarray": ["lodash.castarray@4.4.0", "", {}, "sha512-aVx8ztPv7/2ULbArGJ2Y42bG1mEQ5mGjpdvrbJcJFU3TbYybe+QlLS4pst9zV52ymy2in1KpFPiZnAOATxD4+Q=="],

    "lodash.isplainobject": ["lodash.isplainobject@4.0.6", "", {}, "sha512-oSXzaWypCMHkPC3NvBEaPHf0KsA5mvPrOPgQWDsbg8n7orZ290M0BmC/jgRZ4vcJ6DTAhjrsSYgdsW/F+MFOBA=="],

    "lodash.merge": ["lodash.merge@4.6.2", "", {}, "sha512-0KpjqXRVvrYyCsX1swR/XTK0va6VQkQM6MNo7PqW77ByjAhoARA8EfrP1N4+KlKj8YS0ZUCtRT/YUuhyYDujIQ=="],

    "longest-streak": ["longest-streak@3.1.0", "", {}, "sha512-9Ri+o0JYgehTaVBBDoMqIl8GXtbWg711O3srftcHhZ0dqnETqLaoIK0x17fUw9rFSlK/0NlsKe0Ahhyl5pXE2g=="],

    "loose-envify": ["loose-envify@1.4.0", "", { "dependencies": { "js-tokens": "^3.0.0 || ^4.0.0" }, "bin": "cli.js" }, "sha512-lyuxPGr/Wfhrlem2CL/UcnUc1zcqKAImBDzukY7Y5F/yQiNdko6+fRLevlw1HgMySw7f611UIY408EtxRSoK3Q=="],

    "loupe": ["loupe@3.2.1", "", {}, "sha512-CdzqowRJCeLU72bHvWqwRBBlLcMEtIvGrlvef74kMnV2AolS9Y8xUv1I0U/MNAWMhBlKIoyuEgoJ0t/bbwHbLQ=="],

    "lovable-tagger": ["lovable-tagger@1.1.13", "", { "dependencies": { "esbuild": "^0.25.0", "tailwindcss": "^3.4.17" }, "peerDependencies": { "vite": ">=5.0.0 <8.0.0" } }, "sha512-RBEYDxao7Xf8ya29L0cd+ocE7Gs80xPOIOwwck65Hoie8YDKViuXi3UYV14DoNWIvaJ7WVPf7SG3cc844nFqGA=="],

    "lru-cache": ["lru-cache@10.4.3", "", {}, "sha512-JNAzZcXrCt42VGLuYz0zfAzDfAvJWW6AfYlDBQyDV5DClI2m5sAmK+OIO7s59XfsRsWHp02jAJrRadPRGTt6SQ=="],

    "lucide-react": ["lucide-react@0.462.0", "", { "peerDependencies": { "react": "^16.5.1 || ^17.0.0 || ^18.0.0 || ^19.0.0-rc" } }, "sha512-NTL7EbAao9IFtuSivSZgrAh4fZd09Lr+6MTkqIxuHaH2nnYiYIzXPo06cOxHg9wKLdj6LL8TByG4qpePqwgx/g=="],

    "magic-string": ["magic-string@0.30.21", "", { "dependencies": { "@jridgewell/sourcemap-codec": "^1.5.5" } }, "sha512-vd2F4YUyEXKGcLHoq+TEyCjxueSeHnFxyyjNp80yg0XV4vUhnDer/lvvlqM/arB5bXQN5K2/3oinyCRyx8T2CQ=="],

    "math-intrinsics": ["math-intrinsics@1.1.0", "", {}, "sha512-/IXtbwEk5HTPyEwyKX6hGkYXxM9nbj64B+ilVJnC/R6B0pH5G4V3b0pVbL7DBj4tkhBAppbQUlf6F6Xl9LHu1g=="],

    "mdast-util-from-markdown": ["mdast-util-from-markdown@2.0.3", "", { "dependencies": { "@types/mdast": "^4.0.0", "@types/unist": "^3.0.0", "decode-named-character-reference": "^1.0.0", "devlop": "^1.0.0", "mdast-util-to-string": "^4.0.0", "micromark": "^4.0.0", "micromark-util-decode-numeric-character-reference": "^2.0.0", "micromark-util-decode-string": "^2.0.0", "micromark-util-normalize-identifier": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0", "unist-util-stringify-position": "^4.0.0" } }, "sha512-W4mAWTvSlKvf8L6J+VN9yLSqQ9AOAAvHuoDAmPkz4dHf553m5gVj2ejadHJhoJmcmxEnOv6Pa8XJhpxE93kb8Q=="],

    "mdast-util-mdx-expression": ["mdast-util-mdx-expression@2.0.1", "", { "dependencies": { "@types/estree-jsx": "^1.0.0", "@types/hast": "^3.0.0", "@types/mdast": "^4.0.0", "devlop": "^1.0.0", "mdast-util-from-markdown": "^2.0.0", "mdast-util-to-markdown": "^2.0.0" } }, "sha512-J6f+9hUp+ldTZqKRSg7Vw5V6MqjATc+3E4gf3CFNcuZNWD8XdyI6zQ8GqH7f8169MM6P7hMBRDVGnn7oHB9kXQ=="],

    "mdast-util-mdx-jsx": ["mdast-util-mdx-jsx@3.2.0", "", { "dependencies": { "@types/estree-jsx": "^1.0.0", "@types/hast": "^3.0.0", "@types/mdast": "^4.0.0", "@types/unist": "^3.0.0", "ccount": "^2.0.0", "devlop": "^1.1.0", "mdast-util-from-markdown": "^2.0.0", "mdast-util-to-markdown": "^2.0.0", "parse-entities": "^4.0.0", "stringify-entities": "^4.0.0", "unist-util-stringify-position": "^4.0.0", "vfile-message": "^4.0.0" } }, "sha512-lj/z8v0r6ZtsN/cGNNtemmmfoLAFZnjMbNyLzBafjzikOM+glrjNHPlf6lQDOTccj9n5b0PPihEBbhneMyGs1Q=="],

    "mdast-util-mdxjs-esm": ["mdast-util-mdxjs-esm@2.0.1", "", { "dependencies": { "@types/estree-jsx": "^1.0.0", "@types/hast": "^3.0.0", "@types/mdast": "^4.0.0", "devlop": "^1.0.0", "mdast-util-from-markdown": "^2.0.0", "mdast-util-to-markdown": "^2.0.0" } }, "sha512-EcmOpxsZ96CvlP03NghtH1EsLtr0n9Tm4lPUJUBccV9RwUOneqSycg19n5HGzCf+10LozMRSObtVr3ee1WoHtg=="],

    "mdast-util-phrasing": ["mdast-util-phrasing@4.1.0", "", { "dependencies": { "@types/mdast": "^4.0.0", "unist-util-is": "^6.0.0" } }, "sha512-TqICwyvJJpBwvGAMZjj4J2n0X8QWp21b9l0o7eXyVJ25YNWYbJDVIyD1bZXE6WtV6RmKJVYmQAKWa0zWOABz2w=="],

    "mdast-util-to-hast": ["mdast-util-to-hast@13.2.1", "", { "dependencies": { "@types/hast": "^3.0.0", "@types/mdast": "^4.0.0", "@ungap/structured-clone": "^1.0.0", "devlop": "^1.0.0", "micromark-util-sanitize-uri": "^2.0.0", "trim-lines": "^3.0.0", "unist-util-position": "^5.0.0", "unist-util-visit": "^5.0.0", "vfile": "^6.0.0" } }, "sha512-cctsq2wp5vTsLIcaymblUriiTcZd0CwWtCbLvrOzYCDZoWyMNV8sZ7krj09FSnsiJi3WVsHLM4k6Dq/yaPyCXA=="],

    "mdast-util-to-markdown": ["mdast-util-to-markdown@2.1.2", "", { "dependencies": { "@types/mdast": "^4.0.0", "@types/unist": "^3.0.0", "longest-streak": "^3.0.0", "mdast-util-phrasing": "^4.0.0", "mdast-util-to-string": "^4.0.0", "micromark-util-classify-character": "^2.0.0", "micromark-util-decode-string": "^2.0.0", "unist-util-visit": "^5.0.0", "zwitch": "^2.0.0" } }, "sha512-xj68wMTvGXVOKonmog6LwyJKrYXZPvlwabaryTjLh9LuvovB/KAH+kvi8Gjj+7rJjsFi23nkUxRQv1KqSroMqA=="],

    "mdast-util-to-string": ["mdast-util-to-string@4.0.0", "", { "dependencies": { "@types/mdast": "^4.0.0" } }, "sha512-0H44vDimn51F0YwvxSJSm0eCDOJTRlmN0R1yBh4HLj9wiV1Dn0QoXGbvFAWj2hSItVTlCmBF1hqKlIyUBVFLPg=="],

    "merge2": ["merge2@1.4.1", "", {}, "sha512-8q7VEgMJW4J8tcfVPy8g09NcQwZdbwFEqhe/WZkoIzjn/3TGDwtOCYtXGxA3O8tPzpczCCDgv+P2P5y00ZJOOg=="],

    "micromark": ["micromark@4.0.2", "", { "dependencies": { "@types/debug": "^4.0.0", "debug": "^4.0.0", "decode-named-character-reference": "^1.0.0", "devlop": "^1.0.0", "micromark-core-commonmark": "^2.0.0", "micromark-factory-space": "^2.0.0", "micromark-util-character": "^2.0.0", "micromark-util-chunked": "^2.0.0", "micromark-util-combine-extensions": "^2.0.0", "micromark-util-decode-numeric-character-reference": "^2.0.0", "micromark-util-encode": "^2.0.0", "micromark-util-normalize-identifier": "^2.0.0", "micromark-util-resolve-all": "^2.0.0", "micromark-util-sanitize-uri": "^2.0.0", "micromark-util-subtokenize": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-zpe98Q6kvavpCr1NPVSCMebCKfD7CA2NqZ+rykeNhONIJBpc1tFKt9hucLGwha3jNTNI8lHpctWJWoimVF4PfA=="],

    "micromark-core-commonmark": ["micromark-core-commonmark@2.0.3", "", { "dependencies": { "decode-named-character-reference": "^1.0.0", "devlop": "^1.0.0", "micromark-factory-destination": "^2.0.0", "micromark-factory-label": "^2.0.0", "micromark-factory-space": "^2.0.0", "micromark-factory-title": "^2.0.0", "micromark-factory-whitespace": "^2.0.0", "micromark-util-character": "^2.0.0", "micromark-util-chunked": "^2.0.0", "micromark-util-classify-character": "^2.0.0", "micromark-util-html-tag-name": "^2.0.0", "micromark-util-normalize-identifier": "^2.0.0", "micromark-util-resolve-all": "^2.0.0", "micromark-util-subtokenize": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-RDBrHEMSxVFLg6xvnXmb1Ayr2WzLAWjeSATAoxwKYJV94TeNavgoIdA0a9ytzDSVzBy2YKFK+emCPOEibLeCrg=="],

    "micromark-factory-destination": ["micromark-factory-destination@2.0.1", "", { "dependencies": { "micromark-util-character": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-Xe6rDdJlkmbFRExpTOmRj9N3MaWmbAgdpSrBQvCFqhezUn4AHqJHbaEnfbVYYiexVSs//tqOdY/DxhjdCiJnIA=="],

    "micromark-factory-label": ["micromark-factory-label@2.0.1", "", { "dependencies": { "devlop": "^1.0.0", "micromark-util-character": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-VFMekyQExqIW7xIChcXn4ok29YE3rnuyveW3wZQWWqF4Nv9Wk5rgJ99KzPvHjkmPXF93FXIbBp6YdW3t71/7Vg=="],

    "micromark-factory-space": ["micromark-factory-space@2.0.1", "", { "dependencies": { "micromark-util-character": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-zRkxjtBxxLd2Sc0d+fbnEunsTj46SWXgXciZmHq0kDYGnck/ZSGj9/wULTV95uoeYiK5hRXP2mJ98Uo4cq/LQg=="],

    "micromark-factory-title": ["micromark-factory-title@2.0.1", "", { "dependencies": { "micromark-factory-space": "^2.0.0", "micromark-util-character": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-5bZ+3CjhAd9eChYTHsjy6TGxpOFSKgKKJPJxr293jTbfry2KDoWkhBb6TcPVB4NmzaPhMs1Frm9AZH7OD4Cjzw=="],

    "micromark-factory-whitespace": ["micromark-factory-whitespace@2.0.1", "", { "dependencies": { "micromark-factory-space": "^2.0.0", "micromark-util-character": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-Ob0nuZ3PKt/n0hORHyvoD9uZhr+Za8sFoP+OnMcnWK5lngSzALgQYKMr9RJVOWLqQYuyn6ulqGWSXdwf6F80lQ=="],

    "micromark-util-character": ["micromark-util-character@2.1.1", "", { "dependencies": { "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-wv8tdUTJ3thSFFFJKtpYKOYiGP2+v96Hvk4Tu8KpCAsTMs6yi+nVmGh1syvSCsaxz45J6Jbw+9DD6g97+NV67Q=="],

    "micromark-util-chunked": ["micromark-util-chunked@2.0.1", "", { "dependencies": { "micromark-util-symbol": "^2.0.0" } }, "sha512-QUNFEOPELfmvv+4xiNg2sRYeS/P84pTW0TCgP5zc9FpXetHY0ab7SxKyAQCNCc1eK0459uoLI1y5oO5Vc1dbhA=="],

    "micromark-util-classify-character": ["micromark-util-classify-character@2.0.1", "", { "dependencies": { "micromark-util-character": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-K0kHzM6afW/MbeWYWLjoHQv1sgg2Q9EccHEDzSkxiP/EaagNzCm7T/WMKZ3rjMbvIpvBiZgwR3dKMygtA4mG1Q=="],

    "micromark-util-combine-extensions": ["micromark-util-combine-extensions@2.0.1", "", { "dependencies": { "micromark-util-chunked": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-OnAnH8Ujmy59JcyZw8JSbK9cGpdVY44NKgSM7E9Eh7DiLS2E9RNQf0dONaGDzEG9yjEl5hcqeIsj4hfRkLH/Bg=="],

    "micromark-util-decode-numeric-character-reference": ["micromark-util-decode-numeric-character-reference@2.0.2", "", { "dependencies": { "micromark-util-symbol": "^2.0.0" } }, "sha512-ccUbYk6CwVdkmCQMyr64dXz42EfHGkPQlBj5p7YVGzq8I7CtjXZJrubAYezf7Rp+bjPseiROqe7G6foFd+lEuw=="],

    "micromark-util-decode-string": ["micromark-util-decode-string@2.0.1", "", { "dependencies": { "decode-named-character-reference": "^1.0.0", "micromark-util-character": "^2.0.0", "micromark-util-decode-numeric-character-reference": "^2.0.0", "micromark-util-symbol": "^2.0.0" } }, "sha512-nDV/77Fj6eH1ynwscYTOsbK7rR//Uj0bZXBwJZRfaLEJ1iGBR6kIfNmlNqaqJf649EP0F3NWNdeJi03elllNUQ=="],

    "micromark-util-encode": ["micromark-util-encode@2.0.1", "", {}, "sha512-c3cVx2y4KqUnwopcO9b/SCdo2O67LwJJ/UyqGfbigahfegL9myoEFoDYZgkT7f36T0bLrM9hZTAaAyH+PCAXjw=="],

    "micromark-util-html-tag-name": ["micromark-util-html-tag-name@2.0.1", "", {}, "sha512-2cNEiYDhCWKI+Gs9T0Tiysk136SnR13hhO8yW6BGNyhOC4qYFnwF1nKfD3HFAIXA5c45RrIG1ub11GiXeYd1xA=="],

    "micromark-util-normalize-identifier": ["micromark-util-normalize-identifier@2.0.1", "", { "dependencies": { "micromark-util-symbol": "^2.0.0" } }, "sha512-sxPqmo70LyARJs0w2UclACPUUEqltCkJ6PhKdMIDuJ3gSf/Q+/GIe3WKl0Ijb/GyH9lOpUkRAO2wp0GVkLvS9Q=="],

    "micromark-util-resolve-all": ["micromark-util-resolve-all@2.0.1", "", { "dependencies": { "micromark-util-types": "^2.0.0" } }, "sha512-VdQyxFWFT2/FGJgwQnJYbe1jjQoNTS4RjglmSjTUlpUMa95Htx9NHeYW4rGDJzbjvCsl9eLjMQwGeElsqmzcHg=="],

    "micromark-util-sanitize-uri": ["micromark-util-sanitize-uri@2.0.1", "", { "dependencies": { "micromark-util-character": "^2.0.0", "micromark-util-encode": "^2.0.0", "micromark-util-symbol": "^2.0.0" } }, "sha512-9N9IomZ/YuGGZZmQec1MbgxtlgougxTodVwDzzEouPKo3qFWvymFHWcnDi2vzV1ff6kas9ucW+o3yzJK9YB1AQ=="],

    "micromark-util-subtokenize": ["micromark-util-subtokenize@2.1.0", "", { "dependencies": { "devlop": "^1.0.0", "micromark-util-chunked": "^2.0.0", "micromark-util-symbol": "^2.0.0", "micromark-util-types": "^2.0.0" } }, "sha512-XQLu552iSctvnEcgXw6+Sx75GflAPNED1qx7eBJ+wydBb2KCbRZe+NwvIEEMM83uml1+2WSXpBAcp9IUCgCYWA=="],

    "micromark-util-symbol": ["micromark-util-symbol@2.0.1", "", {}, "sha512-vs5t8Apaud9N28kgCrRUdEed4UJ+wWNvicHLPxCa9ENlYuAY31M0ETy5y1vA33YoNPDFTghEbnh6efaE8h4x0Q=="],

    "micromark-util-types": ["micromark-util-types@2.0.2", "", {}, "sha512-Yw0ECSpJoViF1qTU4DC6NwtC4aWGt1EkzaQB8KPPyCRR8z9TWeV0HbEFGTO+ZY1wB22zmxnJqhPyTpOVCpeHTA=="],

    "micromatch": ["micromatch@4.0.8", "", { "dependencies": { "braces": "^3.0.3", "picomatch": "^2.3.1" } }, "sha512-PXwfBhYu0hBCPw8Dn0E+WDYb7af3dSLVWKi3HGv84IdF4TyFoC0ysxFd0Goxw7nSv4T/PzEJQxsYsEiFCKo2BA=="],

    "mime-db": ["mime-db@1.52.0", "", {}, "sha512-sPU4uV7dYlvtWJxwwxHD0PuihVNiE7TyAbQ5SWxDCB9mUYvOgroQOwYQQOKPJ8CIbE+1ETVlOoK1UC2nU3gYvg=="],

    "mime-types": ["mime-types@2.1.35", "", { "dependencies": { "mime-db": "1.52.0" } }, "sha512-ZDY+bPm5zTTF+YpCrAU9nK0UgICYPT0QtT1NZWFv4s++TNkcgVaT0g6+4R2uI4MjQjzysHB1zxuWL50hzaeXiw=="],

    "min-indent": ["min-indent@1.0.1", "", {}, "sha512-I9jwMn07Sy/IwOj3zVkVik2JTvgpaykDZEigL6Rx6N9LbMywwUSMtxET+7lVoDLLd3O3IXwJwvuuns8UB/HeAg=="],

    "minimatch": ["minimatch@3.1.2", "", { "dependencies": { "brace-expansion": "^1.1.7" } }, "sha512-J7p63hRiAjw1NDEww1W7i37+ByIrOWO5XQQAzZ3VOcL0PNybwpfmV/N05zFAzwQ9USyEcX6t3UO+K5aqBQOIHw=="],

    "minipass": ["minipass@7.1.2", "", {}, "sha512-qOOzS1cBTWYF4BH8fVePDBOO9iptMnGUEZwNc/cMWnTV2nVLZ7VoNWEPHkYczZA0pdoA7dl6e7FL659nX9S2aw=="],

    "motion-dom": ["motion-dom@12.35.0", "", { "dependencies": { "motion-utils": "^12.29.2" } }, "sha512-FFMLEnIejK/zDABn+vqGVAUN4T0+3fw+cVAY8MMT65yR+j5uMuvWdd4npACWhh94OVWQs79CrBBuwOwGRZAQiA=="],

    "motion-utils": ["motion-utils@12.29.2", "", {}, "sha512-G3kc34H2cX2gI63RqU+cZq+zWRRPSsNIOjpdl9TN4AQwC4sgwYPl/Q/Obf/d53nOm569T0fYK+tcoSV50BWx8A=="],

    "ms": ["ms@2.1.3", "", {}, "sha512-6FlzubTLZG3J2a/NVCAleEhjzq5oxgHyaCU9yYXvcLsvoVaHJq/s5xXI6/XXP6tz7R9xAOtHnSO/tXtF3WRTlA=="],

    "mz": ["mz@2.7.0", "", { "dependencies": { "any-promise": "^1.0.0", "object-assign": "^4.0.1", "thenify-all": "^1.0.0" } }, "sha512-z81GNO7nnYMEhrGh9LeymoE4+Yr0Wn5McHIZMK5cfQCl+NDX08sCZgUc9/6MHni9IWuFLm1Z3HTCXu2z9fN62Q=="],

    "nanoid": ["nanoid@3.3.11", "", { "bin": "bin/nanoid.cjs" }, "sha512-N8SpfPUnUp1bK+PMYW8qSWdl9U+wwNWI4QKxOYDy9JAro3WMX7p2OeVRF9v+347pnakNevPmiHhNmZ2HbFA76w=="],

    "natural-compare": ["natural-compare@1.4.0", "", {}, "sha512-OWND8ei3VtNC9h7V60qff3SVobHr996CTwgxubgyQYEpg290h9J0buyECNNJexkFm5sOajh5G116RYA1c8ZMSw=="],

    "next-themes": ["next-themes@0.3.0", "", { "peerDependencies": { "react": "^16.8 || ^17 || ^18", "react-dom": "^16.8 || ^17 || ^18" } }, "sha512-/QHIrsYpd6Kfk7xakK4svpDI5mmXP0gfvCoJdGpZQ2TOrQZmsW0QxjaiLn8wbIKjtm4BTSqLoix4lxYYOnLJ/w=="],

    "node-releases": ["node-releases@2.0.19", "", {}, "sha512-xxOWJsBKtzAq7DY0J+DTzuz58K8e7sJbdgwkbMWQe8UYB6ekmsQ45q0M/tJDsGaZmbC+l7n57UV8Hl5tHxO9uw=="],

    "normalize-path": ["normalize-path@3.0.0", "", {}, "sha512-6eZs5Ls3WtCisHWp9S2GUy8dqkpGi4BVSz3GaqiE6ezub0512ESztXUwUB6C6IKbQkY2Pnb/mD4WYojCRwcwLA=="],

    "normalize-range": ["normalize-range@0.1.2", "", {}, "sha512-bdok/XvKII3nUpklnV6P2hxtMNrCboOjAcyBuQnWEhO665FwrSNRxU+AqpsyvO6LgGYPspN+lu5CLtw4jPRKNA=="],

    "nwsapi": ["nwsapi@2.2.23", "", {}, "sha512-7wfH4sLbt4M0gCDzGE6vzQBo0bfTKjU7Sfpqy/7gs1qBfYz2vEJH6vXcBKpO3+6Yu1telwd0t9HpyOoLEQQbIQ=="],

    "object-assign": ["object-assign@4.1.1", "", {}, "sha512-rJgTQnkUnH1sFw8yT6VSU3zD3sWmu6sZhIseY8VX+GRu3P6F7Fu+JNDoXfklElbLJSnc3FUQHVe4cU5hj+BcUg=="],

    "object-hash": ["object-hash@3.0.0", "", {}, "sha512-RSn9F68PjH9HqtltsSnqYC1XXoWe9Bju5+213R98cNGttag9q9yAOTzdbsqvIa7aNm5WffBZFpWYr2aWrklWAw=="],

    "optionator": ["optionator@0.9.4", "", { "dependencies": { "deep-is": "^0.1.3", "fast-levenshtein": "^2.0.6", "levn": "^0.4.1", "prelude-ls": "^1.2.1", "type-check": "^0.4.0", "word-wrap": "^1.2.5" } }, "sha512-6IpQ7mKUxRcZNLIObR0hz7lxsapSSIYNZJwXPGeF0mTVqGKFIXj1DQcMoT22S3ROcLyY/rz0PWaWZ9ayWmad9g=="],

    "p-limit": ["p-limit@3.1.0", "", { "dependencies": { "yocto-queue": "^0.1.0" } }, "sha512-TYOanM3wGwNGsZN2cVTYPArw454xnXj5qmWF1bEoAc4+cU/ol7GVh7odevjp1FNHduHc3KZMcFduxU5Xc6uJRQ=="],

    "p-locate": ["p-locate@5.0.0", "", { "dependencies": { "p-limit": "^3.0.2" } }, "sha512-LaNjtRWUBY++zB5nE/NwcaoMylSPk+S+ZHNB1TzdbMJMny6dynpAGt7X/tl/QYq3TIeE6nxHppbo2LGymrG5Pw=="],

    "package-json-from-dist": ["package-json-from-dist@1.0.1", "", {}, "sha512-UEZIS3/by4OC8vL3P2dTXRETpebLI2NiI5vIrjaD/5UtrkFX/tNbwjTSRAGC/+7CAo2pIcBaRgWmcBBHcsaCIw=="],

    "parent-module": ["parent-module@1.0.1", "", { "dependencies": { "callsites": "^3.0.0" } }, "sha512-GQ2EWRpQV8/o+Aw8YqtfZZPfNRWZYkbidE9k5rpl/hC3vtHHBfGm2Ifi6qWV+coDGkrUKZAxE3Lot5kcsRlh+g=="],

    "parse-entities": ["parse-entities@4.0.2", "", { "dependencies": { "@types/unist": "^2.0.0", "character-entities-legacy": "^3.0.0", "character-reference-invalid": "^2.0.0", "decode-named-character-reference": "^1.0.0", "is-alphanumerical": "^2.0.0", "is-decimal": "^2.0.0", "is-hexadecimal": "^2.0.0" } }, "sha512-GG2AQYWoLgL877gQIKeRPGO1xF9+eG1ujIb5soS5gPvLQ1y2o8FL90w2QWNdf9I361Mpp7726c+lj3U0qK1uGw=="],

    "parse5": ["parse5@7.3.0", "", { "dependencies": { "entities": "^6.0.0" } }, "sha512-IInvU7fabl34qmi9gY8XOVxhYyMyuH2xUNpb2q8/Y+7552KlejkRvqvD19nMoUW/uQGGbqNpA6Tufu5FL5BZgw=="],

    "path-exists": ["path-exists@4.0.0", "", {}, "sha512-ak9Qy5Q7jYb2Wwcey5Fpvg2KoAc/ZIhLSLOSBmRmygPsGwkVVt0fZa0qrtMz+m6tJTAHfZQ8FnmB4MG4LWy7/w=="],

    "path-key": ["path-key@3.1.1", "", {}, "sha512-ojmeN0qd+y0jszEtoY48r0Peq5dwMEkIlCOu6Q5f41lfkswXuKtYrhgoTpLnyIcHm24Uhqx+5Tqm2InSwLhE6Q=="],

    "path-parse": ["path-parse@1.0.7", "", {}, "sha512-LDJzPVEEEPR+y48z93A0Ed0yXb8pAByGWo/k5YYdYgpY2/2EsOsksJrq7lOHxryrVOn1ejG6oAp8ahvOIQD8sw=="],

    "path-scurry": ["path-scurry@1.11.1", "", { "dependencies": { "lru-cache": "^10.2.0", "minipass": "^5.0.0 || ^6.0.2 || ^7.0.0" } }, "sha512-Xa4Nw17FS9ApQFJ9umLiJS4orGjm7ZzwUrwamcGQuHSzDyth9boKDaycYdDcZDuqYATXw4HFXgaqWTctW/v1HA=="],

    "pathe": ["pathe@2.0.3", "", {}, "sha512-WUjGcAqP1gQacoQe+OBJsFA7Ld4DyXuUIjZ5cc75cLHvJ7dtNsTugphxIADwspS+AraAUePCKrSVtPLFj/F88w=="],

    "pathval": ["pathval@2.0.1", "", {}, "sha512-//nshmD55c46FuFw26xV/xFAaB5HF9Xdap7HJBBnrKdAd6/GxDBaNA1870O79+9ueg61cZLSVc+OaFlfmObYVQ=="],

    "picocolors": ["picocolors@1.1.1", "", {}, "sha512-xceH2snhtb5M9liqDsmEw56le376mTZkEX/jEb/RxNFyegNul7eNslCXP9FDj/Lcu0X8KEyMceP2ntpaHrDEVA=="],

    "picomatch": ["picomatch@4.0.3", "", {}, "sha512-5gTmgEY/sqK6gFXLIsQNH19lWb4ebPDLA4SdLP7dsWkIXHWlG66oPuVvXSGFPppYZz8ZDZq0dYYrbHfBCVUb1Q=="],

    "pify": ["pify@2.3.0", "", {}, "sha512-udgsAY+fTnvv7kI7aaxbqwWNb0AHiB0qBO89PZKPkoTmGOgdbrHDKD+0B2X4uTfJ/FT1R09r9gTsjUjNJotuog=="],

    "pirates": ["pirates@4.0.6", "", {}, "sha512-saLsH7WeYYPiD25LDuLRRY/i+6HaPYr6G1OUlN39otzkSTxKnubR9RTxS3/Kk50s1g2JTgFwWQDQyplC5/SHZg=="],

    "postcss": ["postcss@8.5.6", "", { "dependencies": { "nanoid": "^3.3.11", "picocolors": "^1.1.1", "source-map-js": "^1.2.1" } }, "sha512-3Ybi1tAuwAP9s0r1UQ2J4n5Y0G05bJkpUIO0/bI9MhwmD70S5aTWbXGBwxHrelT+XM1k6dM0pk+SwNkpTRN7Pg=="],

    "postcss-import": ["postcss-import@15.1.0", "", { "dependencies": { "postcss-value-parser": "^4.0.0", "read-cache": "^1.0.0", "resolve": "^1.1.7" }, "peerDependencies": { "postcss": "^8.0.0" } }, "sha512-hpr+J05B2FVYUAXHeK1YyI267J/dDDhMU6B6civm8hSY1jYJnBXxzKDKDswzJmtLHryrjhnDjqqp/49t8FALew=="],

    "postcss-js": ["postcss-js@4.0.1", "", { "dependencies": { "camelcase-css": "^2.0.1" }, "peerDependencies": { "postcss": "^8.4.21" } }, "sha512-dDLF8pEO191hJMtlHFPRa8xsizHaM82MLfNkUHdUtVEV3tgTp5oj+8qbEqYM57SLfc74KSbw//4SeJma2LRVIw=="],

    "postcss-load-config": ["postcss-load-config@4.0.2", "", { "dependencies": { "lilconfig": "^3.0.0", "yaml": "^2.3.4" }, "peerDependencies": { "postcss": ">=8.0.9", "ts-node": ">=9.0.0" }, "optionalPeers": ["ts-node"] }, "sha512-bSVhyJGL00wMVoPUzAVAnbEoWyqRxkjv64tUl427SKnPrENtq6hJwUojroMz2VB+Q1edmi4IfrAPpami5VVgMQ=="],

    "postcss-nested": ["postcss-nested@6.2.0", "", { "dependencies": { "postcss-selector-parser": "^6.1.1" }, "peerDependencies": { "postcss": "^8.2.14" } }, "sha512-HQbt28KulC5AJzG+cZtj9kvKB93CFCdLvog1WFLf1D+xmMvPGlBstkpTEZfK5+AN9hfJocyBFCNiqyS48bpgzQ=="],

    "postcss-selector-parser": ["postcss-selector-parser@6.0.10", "", { "dependencies": { "cssesc": "^3.0.0", "util-deprecate": "^1.0.2" } }, "sha512-IQ7TZdoaqbT+LCpShg46jnZVlhWD2w6iQYAcYXfHARZ7X1t/UGhhceQDs5X0cGqKvYlHNOuv7Oa1xmb0oQuA3w=="],

    "postcss-value-parser": ["postcss-value-parser@4.2.0", "", {}, "sha512-1NNCs6uurfkVbeXG4S8JFT9t19m45ICnif8zWLd5oPSZ50QnwMfK+H3jv408d4jw/7Bttv5axS5IiHoLaVNHeQ=="],

    "prelude-ls": ["prelude-ls@1.2.1", "", {}, "sha512-vkcDPrRZo1QZLbn5RLGPpg/WmIQ65qoWWhcGKf/b5eplkkarX0m9z8ppCat4mlOqUsWpyNuYgO3VRyrYHSzX5g=="],

    "prop-types": ["prop-types@15.8.1", "", { "dependencies": { "loose-envify": "^1.4.0", "object-assign": "^4.1.1", "react-is": "^16.13.1" } }, "sha512-oj87CgZICdulUohogVAR7AjlC0327U4el4L6eAvOqCeudMDVU0NThNaV+b9Df4dXgSP1gXMTnPdhfe/2qDH5cg=="],

    "property-information": ["property-information@7.1.0", "", {}, "sha512-TwEZ+X+yCJmYfL7TPUOcvBZ4QfoT5YenQiJuX//0th53DE6w0xxLEtfK3iyryQFddXuvkIk51EEgrJQ0WJkOmQ=="],

    "psl": ["psl@1.15.0", "", { "dependencies": { "punycode": "^2.3.1" } }, "sha512-JZd3gMVBAVQkSs6HdNZo9Sdo0LNcQeMNP3CozBJb3JYC/QUYZTnKxP+f8oWRX4rHP5EurWxqAHTSwUCjlNKa1w=="],

    "punycode": ["punycode@2.3.1", "", {}, "sha512-vYt7UD1U9Wg6138shLtLOvdAu+8DsC/ilFtEVHcH+wydcSpNE20AfSOduf6MkRFahL5FY7X1oU7nKVZFtfq8Fg=="],

    "querystringify": ["querystringify@2.2.0", "", {}, "sha512-FIqgj2EUvTa7R50u0rGsyTftzjYmv/a3hO345bZNrqabNqjtgiDMgmo4mkUjd+nzU5oF3dClKqFIPUKybUyqoQ=="],

    "queue-microtask": ["queue-microtask@1.2.3", "", {}, "sha512-NuaNSa6flKT5JaSYQzJok04JzTL1CA6aGhv5rfLW3PgqA+M2ChpZQnAC8h8i4ZFkBS8X5RqkDBHA7r4hej3K9A=="],

    "react": ["react@18.3.1", "", { "dependencies": { "loose-envify": "^1.1.0" } }, "sha512-wS+hAgJShR0KhEvPJArfuPVN1+Hz1t0Y6n5jLrGQbkb4urgPE/0Rve+1kMB1v/oWgHgm4WIcV+i7F2pTVj+2iQ=="],

    "react-day-picker": ["react-day-picker@8.10.1", "", { "peerDependencies": { "date-fns": "^2.28.0 || ^3.0.0", "react": "^16.8.0 || ^17.0.0 || ^18.0.0" } }, "sha512-TMx7fNbhLk15eqcMt+7Z7S2KF7mfTId/XJDjKE8f+IUcFn0l08/kI4FiYTL/0yuOLmEcbR4Fwe3GJf/NiiMnPA=="],

    "react-dom": ["react-dom@18.3.1", "", { "dependencies": { "loose-envify": "^1.1.0", "scheduler": "^0.23.2" }, "peerDependencies": { "react": "^18.3.1" } }, "sha512-5m4nQKp+rZRb09LNH59GM4BxTh9251/ylbKIbpe7TpGxfJ+9kv6BLkLBXIjjspbgbnIBNqlI23tRnTWT0snUIw=="],

    "react-hook-form": ["react-hook-form@7.61.1", "", { "peerDependencies": { "react": "^16.8.0 || ^17 || ^18 || ^19" } }, "sha512-2vbXUFDYgqEgM2RcXcAT2PwDW/80QARi+PKmHy5q2KhuKvOlG8iIYgf7eIlIANR5trW9fJbP4r5aub3a4egsew=="],

    "react-is": ["react-is@18.3.1", "", {}, "sha512-/LLMVyas0ljjAtoYiPqYiL8VWXzUUdThrmU5+n20DZv+a+ClRoevUzw5JxU+Ieh5/c87ytoTBV9G1FiKfNJdmg=="],

    "react-markdown": ["react-markdown@10.1.0", "", { "dependencies": { "@types/hast": "^3.0.0", "@types/mdast": "^4.0.0", "devlop": "^1.0.0", "hast-util-to-jsx-runtime": "^2.0.0", "html-url-attributes": "^3.0.0", "mdast-util-to-hast": "^13.0.0", "remark-parse": "^11.0.0", "remark-rehype": "^11.0.0", "unified": "^11.0.0", "unist-util-visit": "^5.0.0", "vfile": "^6.0.0" }, "peerDependencies": { "@types/react": ">=18", "react": ">=18" } }, "sha512-qKxVopLT/TyA6BX3Ue5NwabOsAzm0Q7kAPwq6L+wWDwisYs7R8vZ0nRXqq6rkueboxpkjvLGU9fWifiX/ZZFxQ=="],

    "react-remove-scroll": ["react-remove-scroll@2.7.1", "", { "dependencies": { "react-remove-scroll-bar": "^2.3.7", "react-style-singleton": "^2.2.3", "tslib": "^2.1.0", "use-callback-ref": "^1.3.3", "use-sidecar": "^1.1.3" }, "peerDependencies": { "@types/react": "*", "react": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0 || ^19.0.0-rc" } }, "sha512-HpMh8+oahmIdOuS5aFKKY6Pyog+FNaZV/XyJOq7b4YFwsFHe5yYfdbIalI4k3vU2nSDql7YskmUseHsRrJqIPA=="],

    "react-remove-scroll-bar": ["react-remove-scroll-bar@2.3.8", "", { "dependencies": { "react-style-singleton": "^2.2.2", "tslib": "^2.0.0" }, "peerDependencies": { "@types/react": "*", "react": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0" } }, "sha512-9r+yi9+mgU33AKcj6IbT9oRCO78WriSj6t/cF8DWBZJ9aOGPOTEDvdUDz1FwKim7QXWwmHqtdHnRJfhAxEG46Q=="],

    "react-resizable-panels": ["react-resizable-panels@2.1.9", "", { "peerDependencies": { "react": "^16.14.0 || ^17.0.0 || ^18.0.0 || ^19.0.0 || ^19.0.0-rc", "react-dom": "^16.14.0 || ^17.0.0 || ^18.0.0 || ^19.0.0 || ^19.0.0-rc" } }, "sha512-z77+X08YDIrgAes4jl8xhnUu1LNIRp4+E7cv4xHmLOxxUPO/ML7PSrE813b90vj7xvQ1lcf7g2uA9GeMZonjhQ=="],

    "react-router": ["react-router@6.30.1", "", { "dependencies": { "@remix-run/router": "1.23.0" }, "peerDependencies": { "react": ">=16.8" } }, "sha512-X1m21aEmxGXqENEPG3T6u0Th7g0aS4ZmoNynhbs+Cn+q+QGTLt+d5IQ2bHAXKzKcxGJjxACpVbnYQSCRcfxHlQ=="],

    "react-router-dom": ["react-router-dom@6.30.1", "", { "dependencies": { "@remix-run/router": "1.23.0", "react-router": "6.30.1" }, "peerDependencies": { "react": ">=16.8", "react-dom": ">=16.8" } }, "sha512-llKsgOkZdbPU1Eg3zK8lCn+sjD9wMRZZPuzmdWWX5SUs8OFkN5HnFVC0u5KMeMaC9aoancFI/KoLuKPqN+hxHw=="],

    "react-smooth": ["react-smooth@4.0.4", "", { "dependencies": { "fast-equals": "^5.0.1", "prop-types": "^15.8.1", "react-transition-group": "^4.4.5" }, "peerDependencies": { "react": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0", "react-dom": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0" } }, "sha512-gnGKTpYwqL0Iii09gHobNolvX4Kiq4PKx6eWBCYYix+8cdw+cGo3do906l1NBPKkSWx1DghC1dlWG9L2uGd61Q=="],

    "react-style-singleton": ["react-style-singleton@2.2.3", "", { "dependencies": { "get-nonce": "^1.0.0", "tslib": "^2.0.0" }, "peerDependencies": { "@types/react": "*", "react": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0 || ^19.0.0-rc" } }, "sha512-b6jSvxvVnyptAiLjbkWLE/lOnR4lfTtDAl+eUC7RZy+QQWc6wRzIV2CE6xBuMmDxc2qIihtDCZD5NPOFl7fRBQ=="],

    "react-transition-group": ["react-transition-group@4.4.5", "", { "dependencies": { "@babel/runtime": "^7.5.5", "dom-helpers": "^5.0.1", "loose-envify": "^1.4.0", "prop-types": "^15.6.2" }, "peerDependencies": { "react": ">=16.6.0", "react-dom": ">=16.6.0" } }, "sha512-pZcd1MCJoiKiBR2NRxeCRg13uCXbydPnmB4EOeRrY7480qNWO8IIgQG6zlDkm6uRMsURXPuKq0GWtiM59a5Q6g=="],

    "read-cache": ["read-cache@1.0.0", "", { "dependencies": { "pify": "^2.3.0" } }, "sha512-Owdv/Ft7IjOgm/i0xvNDZ1LrRANRfew4b2prF3OWMQLxLfu3bS8FVhCsrSCMK4lR56Y9ya+AThoTpDCTxCmpRA=="],

    "readdirp": ["readdirp@3.6.0", "", { "dependencies": { "picomatch": "^2.2.1" } }, "sha512-hOS089on8RduqdbhvQ5Z37A0ESjsqz6qnRcffsMU3495FuTdqSm+7bhJ29JvIOsBDEEnan5DPu9t3To9VRlMzA=="],

    "recharts": ["recharts@2.15.4", "", { "dependencies": { "clsx": "^2.0.0", "eventemitter3": "^4.0.1", "lodash": "^4.17.21", "react-is": "^18.3.1", "react-smooth": "^4.0.4", "recharts-scale": "^0.4.4", "tiny-invariant": "^1.3.1", "victory-vendor": "^36.6.8" }, "peerDependencies": { "react": "^16.0.0 || ^17.0.0 || ^18.0.0 || ^19.0.0", "react-dom": "^16.0.0 || ^17.0.0 || ^18.0.0 || ^19.0.0" } }, "sha512-UT/q6fwS3c1dHbXv2uFgYJ9BMFHu3fwnd7AYZaEQhXuYQ4hgsxLvsUXzGdKeZrW5xopzDCvuA2N41WJ88I7zIw=="],

    "recharts-scale": ["recharts-scale@0.4.5", "", { "dependencies": { "decimal.js-light": "^2.4.1" } }, "sha512-kivNFO+0OcUNu7jQquLXAxz1FIwZj8nrj+YkOKc5694NbjCvcT6aSZiIzNzd2Kul4o4rTto8QVR9lMNtxD4G1w=="],

    "redent": ["redent@3.0.0", "", { "dependencies": { "indent-string": "^4.0.0", "strip-indent": "^3.0.0" } }, "sha512-6tDA8g98We0zd0GvVeMT9arEOnTw9qM03L9cJXaCjrip1OO764RDBLBfrB4cwzNGDj5OA5ioymC9GkizgWJDUg=="],

    "remark-parse": ["remark-parse@11.0.0", "", { "dependencies": { "@types/mdast": "^4.0.0", "mdast-util-from-markdown": "^2.0.0", "micromark-util-types": "^2.0.0", "unified": "^11.0.0" } }, "sha512-FCxlKLNGknS5ba/1lmpYijMUzX2esxW5xQqjWxw2eHFfS2MSdaHVINFmhjo+qN1WhZhNimq0dZATN9pH0IDrpA=="],

    "remark-rehype": ["remark-rehype@11.1.2", "", { "dependencies": { "@types/hast": "^3.0.0", "@types/mdast": "^4.0.0", "mdast-util-to-hast": "^13.0.0", "unified": "^11.0.0", "vfile": "^6.0.0" } }, "sha512-Dh7l57ianaEoIpzbp0PC9UKAdCSVklD8E5Rpw7ETfbTl3FqcOOgq5q2LVDhgGCkaBv7p24JXikPdvhhmHvKMsw=="],

    "requires-port": ["requires-port@1.0.0", "", {}, "sha512-KigOCHcocU3XODJxsu8i/j8T9tzT4adHiecwORRQ0ZZFcp7ahwXuRU1m+yuO90C5ZUyGeGfocHDI14M3L3yDAQ=="],

    "resolve": ["resolve@1.22.8", "", { "dependencies": { "is-core-module": "^2.13.0", "path-parse": "^1.0.7", "supports-preserve-symlinks-flag": "^1.0.0" }, "bin": "bin/resolve" }, "sha512-oKWePCxqpd6FlLvGV1VU0x7bkPmmCNolxzjMf4NczoDnQcIWrAF+cPtZn5i6n+RfD2d9i0tzpKnG6Yk168yIyw=="],

    "resolve-from": ["resolve-from@4.0.0", "", {}, "sha512-pb/MYmXstAkysRFx8piNI1tGFNQIFA3vkE3Gq4EuA1dF6gHp/+vgZqsCGJapvy8N3Q+4o7FwvquPJcnZ7RYy4g=="],

    "reusify": ["reusify@1.0.4", "", {}, "sha512-U9nH88a3fc/ekCF1l0/UP1IosiuIjyTh7hBvXVMHYgVcfGvt897Xguj2UOLDeI5BG2m7/uwyaLVT6fbtCwTyzw=="],

    "rollup": ["rollup@4.24.0", "", { "dependencies": { "@types/estree": "1.0.6" }, "optionalDependencies": { "@rollup/rollup-android-arm-eabi": "4.24.0", "@rollup/rollup-android-arm64": "4.24.0", "@rollup/rollup-darwin-arm64": "4.24.0", "@rollup/rollup-darwin-x64": "4.24.0", "@rollup/rollup-linux-arm-gnueabihf": "4.24.0", "@rollup/rollup-linux-arm-musleabihf": "4.24.0", "@rollup/rollup-linux-arm64-gnu": "4.24.0", "@rollup/rollup-linux-arm64-musl": "4.24.0", "@rollup/rollup-linux-powerpc64le-gnu": "4.24.0", "@rollup/rollup-linux-riscv64-gnu": "4.24.0", "@rollup/rollup-linux-s390x-gnu": "4.24.0", "@rollup/rollup-linux-x64-gnu": "4.24.0", "@rollup/rollup-linux-x64-musl": "4.24.0", "@rollup/rollup-win32-arm64-msvc": "4.24.0", "@rollup/rollup-win32-ia32-msvc": "4.24.0", "@rollup/rollup-win32-x64-msvc": "4.24.0", "fsevents": "~2.3.2" }, "bin": "dist/bin/rollup" }, "sha512-DOmrlGSXNk1DM0ljiQA+i+o0rSLhtii1je5wgk60j49d1jHT5YYttBv1iWOnYSTG+fZZESUOSNiAl89SIet+Cg=="],

    "run-parallel": ["run-parallel@1.2.0", "", { "dependencies": { "queue-microtask": "^1.2.2" } }, "sha512-5l4VyZR86LZ/lDxZTR6jqL8AFE2S0IFLMP26AbjsLVADxHdhB/c0GUsH+y39UfCi3dzz8OlQuPmnaJOMoDHQBA=="],

    "safer-buffer": ["safer-buffer@2.1.2", "", {}, "sha512-YZo3K82SD7Riyi0E1EQPojLz7kpepnSQI9IyPbHHg1XXXevb5dJI7tpyN2ADxGcQbHG7vcyRHk0cbwqcQriUtg=="],

    "saxes": ["saxes@6.0.0", "", { "dependencies": { "xmlchars": "^2.2.0" } }, "sha512-xAg7SOnEhrm5zI3puOOKyy1OMcMlIJZYNJY7xLBwSze0UjhPLnWfj2GF2EpT0jmzaJKIWKHLsaSSajf35bcYnA=="],

    "scheduler": ["scheduler@0.23.2", "", { "dependencies": { "loose-envify": "^1.1.0" } }, "sha512-UOShsPwz7NrMUqhR6t0hWjFduvOzbtv7toDH1/hIrfRNIDBnnBWd0CwJTGvTpngVlmwGCdP9/Zl/tVrDqcuYzQ=="],

    "semver": ["semver@7.7.2", "", { "bin": "bin/semver.js" }, "sha512-RF0Fw+rO5AMf9MAyaRXI4AV0Ulj5lMHqVxxdSgiVbixSCXoEmmX/jk0CuJw4+3SqroYO9VoUh+HcuJivvtJemA=="],

    "shebang-command": ["shebang-command@2.0.0", "", { "dependencies": { "shebang-regex": "^3.0.0" } }, "sha512-kHxr2zZpYtdmrN1qDjrrX/Z1rR1kG8Dx+gkpK1G4eXmvXswmcE1hTWBWYUzlraYw1/yZp6YuDY77YtvbN0dmDA=="],

    "shebang-regex": ["shebang-regex@3.0.0", "", {}, "sha512-7++dFhtcx3353uBaq8DDR4NuxBetBzC7ZQOhmTQInHEd6bSrXdiEyzCvG07Z44UYdLShWUyXt5M/yhz8ekcb1A=="],

    "siginfo": ["siginfo@2.0.0", "", {}, "sha512-ybx0WO1/8bSBLEWXZvEd7gMW3Sn3JFlW3TvX1nREbDLRNQNaeNN8WK0meBwPdAaOI7TtRRRJn/Es1zhrrCHu7g=="],

    "signal-exit": ["signal-exit@4.1.0", "", {}, "sha512-bzyZ1e88w9O1iNJbKnOlvYTrWPDl46O1bG0D3XInv+9tkPrxrN8jUUTiFlDkkmKWgn1M6CfIA13SuGqOa9Korw=="],

    "sonner": ["sonner@1.7.4", "", { "peerDependencies": { "react": "^18.0.0 || ^19.0.0 || ^19.0.0-rc", "react-dom": "^18.0.0 || ^19.0.0 || ^19.0.0-rc" } }, "sha512-DIS8z4PfJRbIyfVFDVnK9rO3eYDtse4Omcm6bt0oEr5/jtLgysmjuBl1frJ9E/EQZrFmKx2A8m/s5s9CRXIzhw=="],

    "source-map": ["source-map@0.6.1", "", {}, "sha512-UjgapumWlbMhkBgzT7Ykc5YXUT46F0iKu8SGXq0bcwP5dz/h0Plj6enJqjz1Zbq2l5WaqYnrVbwWOWMyF3F47g=="],

    "source-map-js": ["source-map-js@1.2.1", "", {}, "sha512-UXWMKhLOwVKb728IUtQPXxfYU+usdybtUrK/8uGE8CQMvrhOpwvzDBwj0QhSL7MQc7vIsISBG8VQ8+IDQxpfQA=="],

    "space-separated-tokens": ["space-separated-tokens@2.0.2", "", {}, "sha512-PEGlAwrG8yXGXRjW32fGbg66JAlOAwbObuqVoJpv/mRgoWDQfgH1wDPvtzWyUSNAXBGSk8h755YDbbcEy3SH2Q=="],

    "stackback": ["stackback@0.0.2", "", {}, "sha512-1XMJE5fQo1jGH6Y/7ebnwPOBEkIEnT4QF32d5R1+VXdXveM0IBMJt8zfaxX1P3QhVwrYe+576+jkANtSS2mBbw=="],

    "std-env": ["std-env@3.10.0", "", {}, "sha512-5GS12FdOZNliM5mAOxFRg7Ir0pWz8MdpYm6AY6VPkGpbA7ZzmbzNcBJQ0GPvvyWgcY7QAhCgf9Uy89I03faLkg=="],

    "string-width": ["string-width@5.1.2", "", { "dependencies": { "eastasianwidth": "^0.2.0", "emoji-regex": "^9.2.2", "strip-ansi": "^7.0.1" } }, "sha512-HnLOCR3vjcY8beoNLtcjZ5/nxn2afmME6lhrDrebokqMap+XbeW8n9TXpPDOqdGK5qcI3oT0GKTW6wC7EMiVqA=="],

    "string-width-cjs": ["string-width@4.2.3", "", { "dependencies": { "emoji-regex": "^8.0.0", "is-fullwidth-code-point": "^3.0.0", "strip-ansi": "^6.0.1" } }, "sha512-wKyQRQpjJ0sIp62ErSZdGsjMJWsap5oRNihHhu6G7JVO/9jIB6UyevL+tXuOqrng8j/cxKTWyWUwvSTriiZz/g=="],

    "stringify-entities": ["stringify-entities@4.0.4", "", { "dependencies": { "character-entities-html4": "^2.0.0", "character-entities-legacy": "^3.0.0" } }, "sha512-IwfBptatlO+QCJUo19AqvrPNqlVMpW9YEL2LIVY+Rpv2qsjCGxaDLNRgeGsQWJhfItebuJhsGSLjaBbNSQ+ieg=="],

    "strip-ansi": ["strip-ansi@7.1.0", "", { "dependencies": { "ansi-regex": "^6.0.1" } }, "sha512-iq6eVVI64nQQTRYq2KtEg2d2uU7LElhTJwsH4YzIHZshxlgZms/wIc4VoDQTlG/IvVIrBKG06CrZnp0qv7hkcQ=="],

    "strip-ansi-cjs": ["strip-ansi@6.0.1", "", { "dependencies": { "ansi-regex": "^5.0.1" } }, "sha512-Y38VPSHcqkFrCpFnQ9vuSXmquuv5oXOKpGeT6aGrr3o3Gc9AlVa6JBfUSOCnbxGGZF+/0ooI7KrPuUSztUdU5A=="],

    "strip-indent": ["strip-indent@3.0.0", "", { "dependencies": { "min-indent": "^1.0.0" } }, "sha512-laJTa3Jb+VQpaC6DseHhF7dXVqHTfJPCRDaEbid/drOhgitgYku/letMUqOXFoWV0zIIUbjpdH2t+tYj4bQMRQ=="],

    "strip-json-comments": ["strip-json-comments@3.1.1", "", {}, "sha512-6fPc+R4ihwqP6N/aIv2f1gMH8lOVtWQHoqC4yK6oSDVVocumAsfCqjkXnqiYMhmMwS/mEHLp7Vehlt3ql6lEig=="],

    "strip-literal": ["strip-literal@3.1.0", "", { "dependencies": { "js-tokens": "^9.0.1" } }, "sha512-8r3mkIM/2+PpjHoOtiAW8Rg3jJLHaV7xPwG+YRGrv6FP0wwk/toTpATxWYOW0BKdWwl82VT2tFYi5DlROa0Mxg=="],

    "style-to-js": ["style-to-js@1.1.21", "", { "dependencies": { "style-to-object": "1.0.14" } }, "sha512-RjQetxJrrUJLQPHbLku6U/ocGtzyjbJMP9lCNK7Ag0CNh690nSH8woqWH9u16nMjYBAok+i7JO1NP2pOy8IsPQ=="],

    "style-to-object": ["style-to-object@1.0.14", "", { "dependencies": { "inline-style-parser": "0.2.7" } }, "sha512-LIN7rULI0jBscWQYaSswptyderlarFkjQ+t79nzty8tcIAceVomEVlLzH5VP4Cmsv6MtKhs7qaAiwlcp+Mgaxw=="],

    "sucrase": ["sucrase@3.35.0", "", { "dependencies": { "@jridgewell/gen-mapping": "^0.3.2", "commander": "^4.0.0", "glob": "^10.3.10", "lines-and-columns": "^1.1.6", "mz": "^2.7.0", "pirates": "^4.0.1", "ts-interface-checker": "^0.1.9" }, "bin": { "sucrase": "bin/sucrase", "sucrase-node": "bin/sucrase-node" } }, "sha512-8EbVDiu9iN/nESwxeSxDKe0dunta1GOlHufmSSXxMD2z2/tMZpDMpvXQGsc+ajGo8y2uYUmixaSRUc/QPoQ0GA=="],

    "supports-color": ["supports-color@7.2.0", "", { "dependencies": { "has-flag": "^4.0.0" } }, "sha512-qpCAvRl9stuOHveKsn7HncJRvv501qIacKzQlO/+Lwxc9+0q2wLyv4Dfvt80/DPn2pqOBsJdDiogXGR9+OvwRw=="],

    "supports-preserve-symlinks-flag": ["supports-preserve-symlinks-flag@1.0.0", "", {}, "sha512-ot0WnXS9fgdkgIcePe6RHNk1WA8+muPa6cSjeR3V8K27q9BB1rTE3R1p7Hv0z1ZyAc8s6Vvv8DIyWf681MAt0w=="],

    "symbol-tree": ["symbol-tree@3.2.4", "", {}, "sha512-9QNk5KwDF+Bvz+PyObkmSYjI5ksVUYtjW7AU22r2NKcfLJcXp96hkDWU3+XndOsUb+AQ9QhfzfCT2O+CNWT5Tw=="],

    "tailwind-merge": ["tailwind-merge@2.6.0", "", {}, "sha512-P+Vu1qXfzediirmHOC3xKGAYeZtPcV9g76X+xg2FD4tYgR71ewMA35Y3sCz3zhiN/dwefRpJX0yBcgwi1fXNQA=="],

    "tailwindcss": ["tailwindcss@3.4.17", "", { "dependencies": { "@alloc/quick-lru": "^5.2.0", "arg": "^5.0.2", "chokidar": "^3.6.0", "didyoumean": "^1.2.2", "dlv": "^1.1.3", "fast-glob": "^3.3.2", "glob-parent": "^6.0.2", "is-glob": "^4.0.3", "jiti": "^1.21.6", "lilconfig": "^3.1.3", "micromatch": "^4.0.8", "normalize-path": "^3.0.0", "object-hash": "^3.0.0", "picocolors": "^1.1.1", "postcss": "^8.4.47", "postcss-import": "^15.1.0", "postcss-js": "^4.0.1", "postcss-load-config": "^4.0.2", "postcss-nested": "^6.2.0", "postcss-selector-parser": "^6.1.2", "resolve": "^1.22.8", "sucrase": "^3.35.0" }, "bin": { "tailwind": "lib/cli.js", "tailwindcss": "lib/cli.js" } }, "sha512-w33E2aCvSDP0tW9RZuNXadXlkHXqFzSkQew/aIa2i/Sj8fThxwovwlXHSPXTbAHwEIhBFXAedUhP2tueAKP8Og=="],

    "tailwindcss-animate": ["tailwindcss-animate@1.0.7", "", { "peerDependencies": { "tailwindcss": ">=3.0.0 || insiders" } }, "sha512-bl6mpH3T7I3UFxuvDEXLxy/VuFxBk5bbzplh7tXI68mwMokNYd1t9qPBHlnyTwfa4JGC4zP516I1hYYtQ/vspA=="],

    "thenify": ["thenify@3.3.1", "", { "dependencies": { "any-promise": "^1.0.0" } }, "sha512-RVZSIV5IG10Hk3enotrhvz0T9em6cyHBLkH/YAZuKqd8hRkKhSfCGIcP2KUY0EPxndzANBmNllzWPwak+bheSw=="],

    "thenify-all": ["thenify-all@1.6.0", "", { "dependencies": { "thenify": ">= 3.1.0 < 4" } }, "sha512-RNxQH/qI8/t3thXJDwcstUO4zeqo64+Uy/+sNVRBx4Xn2OX+OZ9oP+iJnNFqplFra2ZUVeKCSa2oVWi3T4uVmA=="],

    "tiny-invariant": ["tiny-invariant@1.3.3", "", {}, "sha512-+FbBPE1o9QAYvviau/qC5SE3caw21q3xkvWKBtja5vgqOWIHHJ3ioaq1VPfn/Szqctz2bU/oYeKd9/z5BL+PVg=="],

    "tinybench": ["tinybench@2.9.0", "", {}, "sha512-0+DUvqWMValLmha6lr4kD8iAMK1HzV0/aKnCtWb9v9641TnP/MFb7Pc2bxoxQjTXAErryXVgUOfv2YqNllqGeg=="],

    "tinyexec": ["tinyexec@0.3.2", "", {}, "sha512-KQQR9yN7R5+OSwaK0XQoj22pwHoTlgYqmUscPYoknOoWCWfj/5/ABTMRi69FrKU5ffPVh5QcFikpWJI/P1ocHA=="],

    "tinyglobby": ["tinyglobby@0.2.15", "", { "dependencies": { "fdir": "^6.5.0", "picomatch": "^4.0.3" } }, "sha512-j2Zq4NyQYG5XMST4cbs02Ak8iJUdxRM0XI5QyxXuZOzKOINmWurp3smXu3y5wDcJrptwpSjgXHzIQxR0omXljQ=="],

    "tinypool": ["tinypool@1.1.1", "", {}, "sha512-Zba82s87IFq9A9XmjiX5uZA/ARWDrB03OHlq+Vw1fSdt0I+4/Kutwy8BP4Y/y/aORMo61FQ0vIb5j44vSo5Pkg=="],

    "tinyrainbow": ["tinyrainbow@2.0.0", "", {}, "sha512-op4nsTR47R6p0vMUUoYl/a+ljLFVtlfaXkLQmqfLR1qHma1h/ysYk4hEXZ880bf2CYgTskvTa/e196Vd5dDQXw=="],

    "tinyspy": ["tinyspy@4.0.4", "", {}, "sha512-azl+t0z7pw/z958Gy9svOTuzqIk6xq+NSheJzn5MMWtWTFywIacg2wUlzKFGtt3cthx0r2SxMK0yzJOR0IES7Q=="],

    "to-regex-range": ["to-regex-range@5.0.1", "", { "dependencies": { "is-number": "^7.0.0" } }, "sha512-65P7iz6X5yEr1cwcgvQxbbIw7Uk3gOy5dIdtZ4rDveLqhrdJP+Li/Hx6tyK0NEb+2GCyneCMJiGqrADCSNk8sQ=="],

    "tough-cookie": ["tough-cookie@4.1.4", "", { "dependencies": { "psl": "^1.1.33", "punycode": "^2.1.1", "universalify": "^0.2.0", "url-parse": "^1.5.3" } }, "sha512-Loo5UUvLD9ScZ6jh8beX1T6sO1w2/MpCRpEP7V280GKMVUQ0Jzar2U3UJPsrdbziLEMMhu3Ujnq//rhiFuIeag=="],

    "tr46": ["tr46@3.0.0", "", { "dependencies": { "punycode": "^2.1.1" } }, "sha512-l7FvfAHlcmulp8kr+flpQZmVwtu7nfRV7NZujtN0OqES8EL4O4e0qqzL0DC5gAvx/ZC/9lk6rhcUwYvkBnBnYA=="],

    "trim-lines": ["trim-lines@3.0.1", "", {}, "sha512-kRj8B+YHZCc9kQYdWfJB2/oUl9rA99qbowYYBtr4ui4mZyAQ2JpvVBd/6U2YloATfqBhBTSMhTpgBHtU0Mf3Rg=="],

    "trough": ["trough@2.2.0", "", {}, "sha512-tmMpK00BjZiUyVyvrBK7knerNgmgvcV/KLVyuma/SC+TQN167GrMRciANTz09+k3zW8L8t60jWO1GpfkZdjTaw=="],

    "ts-api-utils": ["ts-api-utils@2.1.0", "", { "peerDependencies": { "typescript": ">=4.8.4" } }, "sha512-CUgTZL1irw8u29bzrOD/nH85jqyc74D6SshFgujOIA7osm2Rz7dYH77agkx7H4FBNxDq7Cjf+IjaX/8zwFW+ZQ=="],

    "ts-interface-checker": ["ts-interface-checker@0.1.13", "", {}, "sha512-Y/arvbn+rrz3JCKl9C4kVNfTfSm2/mEp5FSz5EsZSANGPSlQrpRI5M4PKF+mJnE52jOO90PnPSc3Ur3bTQw0gA=="],

    "tslib": ["tslib@2.8.1", "", {}, "sha512-oJFu94HQb+KVduSUQL7wnpmqnfmLsOA/nAh6b6EH0wCEoK0/mPeXU6c3wKDV83MkOuHPRHtSXKKU99IBazS/2w=="],

    "type-check": ["type-check@0.4.0", "", { "dependencies": { "prelude-ls": "^1.2.1" } }, "sha512-XleUoc9uwGXqjWwXaUTZAmzMcFZ5858QA2vvx1Ur5xIcixXIP+8LnFDgRplU30us6teqdlskFfu+ae4K79Ooew=="],

    "typescript": ["typescript@5.8.3", "", { "bin": { "tsc": "bin/tsc", "tsserver": "bin/tsserver" } }, "sha512-p1diW6TqL9L07nNxvRMM7hMMw4c5XOo/1ibL4aAIGmSAt9slTE1Xgw5KWuof2uTOvCg9BY7ZRi+GaF+7sfgPeQ=="],

    "typescript-eslint": ["typescript-eslint@8.38.0", "", { "dependencies": { "@typescript-eslint/eslint-plugin": "8.38.0", "@typescript-eslint/parser": "8.38.0", "@typescript-eslint/typescript-estree": "8.38.0", "@typescript-eslint/utils": "8.38.0" }, "peerDependencies": { "eslint": "^8.57.0 || ^9.0.0", "typescript": ">=4.8.4 <5.9.0" } }, "sha512-FsZlrYK6bPDGoLeZRuvx2v6qrM03I0U0SnfCLPs/XCCPCFD80xU9Pg09H/K+XFa68uJuZo7l/Xhs+eDRg2l3hg=="],

    "undici-types": ["undici-types@6.21.0", "", {}, "sha512-iwDZqg0QAGrg9Rav5H4n0M64c3mkR59cJ6wQp+7C4nI0gsmExaedaYLNO44eT4AtBBwjbTiGPMlt2Md0T9H9JQ=="],

    "unified": ["unified@11.0.5", "", { "dependencies": { "@types/unist": "^3.0.0", "bail": "^2.0.0", "devlop": "^1.0.0", "extend": "^3.0.0", "is-plain-obj": "^4.0.0", "trough": "^2.0.0", "vfile": "^6.0.0" } }, "sha512-xKvGhPWw3k84Qjh8bI3ZeJjqnyadK+GEFtazSfZv/rKeTkTjOJho6mFqh2SM96iIcZokxiOpg78GazTSg8+KHA=="],

    "unist-util-is": ["unist-util-is@6.0.1", "", { "dependencies": { "@types/unist": "^3.0.0" } }, "sha512-LsiILbtBETkDz8I9p1dQ0uyRUWuaQzd/cuEeS1hoRSyW5E5XGmTzlwY1OrNzzakGowI9Dr/I8HVaw4hTtnxy8g=="],

    "unist-util-position": ["unist-util-position@5.0.0", "", { "dependencies": { "@types/unist": "^3.0.0" } }, "sha512-fucsC7HjXvkB5R3kTCO7kUjRdrS0BJt3M/FPxmHMBOm8JQi2BsHAHFsy27E0EolP8rp0NzXsJ+jNPyDWvOJZPA=="],

    "unist-util-stringify-position": ["unist-util-stringify-position@4.0.0", "", { "dependencies": { "@types/unist": "^3.0.0" } }, "sha512-0ASV06AAoKCDkS2+xw5RXJywruurpbC4JZSm7nr7MOt1ojAzvyyaO+UxZf18j8FCF6kmzCZKcAgN/yu2gm2XgQ=="],

    "unist-util-visit": ["unist-util-visit@5.1.0", "", { "dependencies": { "@types/unist": "^3.0.0", "unist-util-is": "^6.0.0", "unist-util-visit-parents": "^6.0.0" } }, "sha512-m+vIdyeCOpdr/QeQCu2EzxX/ohgS8KbnPDgFni4dQsfSCtpz8UqDyY5GjRru8PDKuYn7Fq19j1CQ+nJSsGKOzg=="],

    "unist-util-visit-parents": ["unist-util-visit-parents@6.0.2", "", { "dependencies": { "@types/unist": "^3.0.0", "unist-util-is": "^6.0.0" } }, "sha512-goh1s1TBrqSqukSc8wrjwWhL0hiJxgA8m4kFxGlQ+8FYQ3C/m11FcTs4YYem7V664AhHVvgoQLk890Ssdsr2IQ=="],

    "universalify": ["universalify@0.2.0", "", {}, "sha512-CJ1QgKmNg3CwvAv/kOFmtnEN05f0D/cn9QntgNOQlQF9dgvVTHj3t+8JPdjqawCHk7V/KA+fbUqzZ9XWhcqPUg=="],

    "update-browserslist-db": ["update-browserslist-db@1.1.3", "", { "dependencies": { "escalade": "^3.2.0", "picocolors": "^1.1.1" }, "peerDependencies": { "browserslist": ">= 4.21.0" }, "bin": "cli.js" }, "sha512-UxhIZQ+QInVdunkDAaiazvvT/+fXL5Osr0JZlJulepYu6Jd7qJtDZjlur0emRlT71EN3ScPoE7gvsuIKKNavKw=="],

    "uri-js": ["uri-js@4.4.1", "", { "dependencies": { "punycode": "^2.1.0" } }, "sha512-7rKUyy33Q1yc98pQ1DAmLtwX109F7TIfWlW1Ydo8Wl1ii1SeHieeh0HHfPeL2fMXK6z0s8ecKs9frCuLJvndBg=="],

    "url-parse": ["url-parse@1.5.10", "", { "dependencies": { "querystringify": "^2.1.1", "requires-port": "^1.0.0" } }, "sha512-WypcfiRhfeUP9vvF0j6rw0J3hrWrw6iZv3+22h6iRMJ/8z1Tj6XfLP4DsUix5MhMPnXpiHDoKyoZ/bdCkwBCiQ=="],

    "use-callback-ref": ["use-callback-ref@1.3.3", "", { "dependencies": { "tslib": "^2.0.0" }, "peerDependencies": { "@types/react": "*", "react": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0 || ^19.0.0-rc" } }, "sha512-jQL3lRnocaFtu3V00JToYz/4QkNWswxijDaCVNZRiRTO3HQDLsdu1ZtmIUvV4yPp+rvWm5j0y0TG/S61cuijTg=="],

    "use-sidecar": ["use-sidecar@1.1.3", "", { "dependencies": { "detect-node-es": "^1.1.0", "tslib": "^2.0.0" }, "peerDependencies": { "@types/react": "*", "react": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0 || ^19.0.0-rc" } }, "sha512-Fedw0aZvkhynoPYlA5WXrMCAMm+nSWdZt6lzJQ7Ok8S6Q+VsHmHpRWndVRJ8Be0ZbkfPc5LRYH+5XrzXcEeLRQ=="],

    "use-sync-external-store": ["use-sync-external-store@1.5.0", "", { "peerDependencies": { "react": "^16.8.0 || ^17.0.0 || ^18.0.0 || ^19.0.0" } }, "sha512-Rb46I4cGGVBmjamjphe8L/UnvJD+uPPtTkNvX5mZgqdbavhI4EbgIWJiIHXJ8bc/i9EQGPRh4DwEURJ552Do0A=="],

    "util-deprecate": ["util-deprecate@1.0.2", "", {}, "sha512-EPD5q1uXyFxJpCrLnCc1nHnq3gOa6DZBocAIiI2TaSCA7VCJ1UJDMagCzIkXNsUYfD1daK//LTEQ8xiIbrHtcw=="],

    "vaul": ["vaul@0.9.9", "", { "dependencies": { "@radix-ui/react-dialog": "^1.1.1" }, "peerDependencies": { "react": "^16.8 || ^17.0 || ^18.0", "react-dom": "^16.8 || ^17.0 || ^18.0" } }, "sha512-7afKg48srluhZwIkaU+lgGtFCUsYBSGOl8vcc8N/M3YQlZFlynHD15AE+pwrYdc826o7nrIND4lL9Y6b9WWZZQ=="],

    "vfile": ["vfile@6.0.3", "", { "dependencies": { "@types/unist": "^3.0.0", "vfile-message": "^4.0.0" } }, "sha512-KzIbH/9tXat2u30jf+smMwFCsno4wHVdNmzFyL+T/L3UGqqk6JKfVqOFOZEpZSHADH1k40ab6NUIXZq422ov3Q=="],

    "vfile-message": ["vfile-message@4.0.3", "", { "dependencies": { "@types/unist": "^3.0.0", "unist-util-stringify-position": "^4.0.0" } }, "sha512-QTHzsGd1EhbZs4AsQ20JX1rC3cOlt/IWJruk893DfLRr57lcnOeMaWG4K0JrRta4mIJZKth2Au3mM3u03/JWKw=="],

    "victory-vendor": ["victory-vendor@36.9.2", "", { "dependencies": { "@types/d3-array": "^3.0.3", "@types/d3-ease": "^3.0.0", "@types/d3-interpolate": "^3.0.1", "@types/d3-scale": "^4.0.2", "@types/d3-shape": "^3.1.0", "@types/d3-time": "^3.0.0", "@types/d3-timer": "^3.0.0", "d3-array": "^3.1.6", "d3-ease": "^3.0.1", "d3-interpolate": "^3.0.1", "d3-scale": "^4.0.2", "d3-shape": "^3.1.0", "d3-time": "^3.0.0", "d3-timer": "^3.0.1" } }, "sha512-PnpQQMuxlwYdocC8fIJqVXvkeViHYzotI+NJrCuav0ZYFoq912ZHBk3mCeuj+5/VpodOjPe1z0Fk2ihgzlXqjQ=="],

    "vite": ["vite@5.4.19", "", { "dependencies": { "esbuild": "^0.21.3", "postcss": "^8.4.43", "rollup": "^4.20.0" }, "optionalDependencies": { "fsevents": "~2.3.3" }, "peerDependencies": { "@types/node": "^18.0.0 || >=20.0.0", "less": "*", "lightningcss": "^1.21.0", "sass": "*", "sass-embedded": "*", "stylus": "*", "sugarss": "*", "terser": "^5.4.0" }, "optionalPeers": ["less", "lightningcss", "sass", "sass-embedded", "stylus", "sugarss", "terser"], "bin": "bin/vite.js" }, "sha512-qO3aKv3HoQC8QKiNSTuUM1l9o/XX3+c+VTgLHbJWHZGeTPVAg2XwazI9UWzoxjIJCGCV2zU60uqMzjeLZuULqA=="],

    "vite-node": ["vite-node@3.2.4", "", { "dependencies": { "cac": "^6.7.14", "debug": "^4.4.1", "es-module-lexer": "^1.7.0", "pathe": "^2.0.3", "vite": "^5.0.0 || ^6.0.0 || ^7.0.0-0" }, "bin": "vite-node.mjs" }, "sha512-EbKSKh+bh1E1IFxeO0pg1n4dvoOTt0UDiXMd/qn++r98+jPO1xtJilvXldeuQ8giIB5IkpjCgMleHMNEsGH6pg=="],

    "vitest": ["vitest@3.2.4", "", { "dependencies": { "@types/chai": "^5.2.2", "@vitest/expect": "3.2.4", "@vitest/mocker": "3.2.4", "@vitest/pretty-format": "^3.2.4", "@vitest/runner": "3.2.4", "@vitest/snapshot": "3.2.4", "@vitest/spy": "3.2.4", "@vitest/utils": "3.2.4", "chai": "^5.2.0", "debug": "^4.4.1", "expect-type": "^1.2.1", "magic-string": "^0.30.17", "pathe": "^2.0.3", "picomatch": "^4.0.2", "std-env": "^3.9.0", "tinybench": "^2.9.0", "tinyexec": "^0.3.2", "tinyglobby": "^0.2.14", "tinypool": "^1.1.1", "tinyrainbow": "^2.0.0", "vite": "^5.0.0 || ^6.0.0 || ^7.0.0-0", "vite-node": "3.2.4", "why-is-node-running": "^2.3.0" }, "peerDependencies": { "@edge-runtime/vm": "*", "@types/debug": "^4.1.12", "@types/node": "^18.0.0 || ^20.0.0 || >=22.0.0", "@vitest/browser": "3.2.4", "@vitest/ui": "3.2.4", "happy-dom": "*", "jsdom": "*" }, "optionalPeers": ["@edge-runtime/vm", "@vitest/browser", "@vitest/ui", "happy-dom"], "bin": "vitest.mjs" }, "sha512-LUCP5ev3GURDysTWiP47wRRUpLKMOfPh+yKTx3kVIEiu5KOMeqzpnYNsKyOoVrULivR8tLcks4+lga33Whn90A=="],

    "w3c-xmlserializer": ["w3c-xmlserializer@4.0.0", "", { "dependencies": { "xml-name-validator": "^4.0.0" } }, "sha512-d+BFHzbiCx6zGfz0HyQ6Rg69w9k19nviJspaj4yNscGjrHu94sVP+aRm75yEbCh+r2/yR+7q6hux9LVtbuTGBw=="],

    "webidl-conversions": ["webidl-conversions@7.0.0", "", {}, "sha512-VwddBukDzu71offAQR975unBIGqfKZpM+8ZX6ySk8nYhVoo5CYaZyzt3YBvYtRtO+aoGlqxPg/B87NGVZ/fu6g=="],

    "whatwg-encoding": ["whatwg-encoding@2.0.0", "", { "dependencies": { "iconv-lite": "0.6.3" } }, "sha512-p41ogyeMUrw3jWclHWTQg1k05DSVXPLcVxRTYsXUk+ZooOCZLcoYgPZ/HL/D/N+uQPOtcp1me1WhBEaX02mhWg=="],

    "whatwg-mimetype": ["whatwg-mimetype@3.0.0", "", {}, "sha512-nt+N2dzIutVRxARx1nghPKGv1xHikU7HKdfafKkLNLindmPU/ch3U31NOCGGA/dmPcmb1VlofO0vnKAcsm0o/Q=="],

    "whatwg-url": ["whatwg-url@11.0.0", "", { "dependencies": { "tr46": "^3.0.0", "webidl-conversions": "^7.0.0" } }, "sha512-RKT8HExMpoYx4igMiVMY83lN6UeITKJlBQ+vR/8ZJ8OCdSiN3RwCq+9gH0+Xzj0+5IrM6i4j/6LuvzbZIQgEcQ=="],

    "which": ["which@2.0.2", "", { "dependencies": { "isexe": "^2.0.0" }, "bin": { "node-which": "bin/node-which" } }, "sha512-BLI3Tl1TW3Pvl70l3yq3Y64i+awpwXqsGBYWkkqMtnbXgrMD+yj7rhW0kuEDxzJaYXGjEW5ogapKNMEKNMjibA=="],

    "why-is-node-running": ["why-is-node-running@2.3.0", "", { "dependencies": { "siginfo": "^2.0.0", "stackback": "0.0.2" }, "bin": "cli.js" }, "sha512-hUrmaWBdVDcxvYqnyh09zunKzROWjbZTiNy8dBEjkS7ehEDQibXJ7XvlmtbwuTclUiIyN+CyXQD4Vmko8fNm8w=="],

    "word-wrap": ["word-wrap@1.2.5", "", {}, "sha512-BN22B5eaMMI9UMtjrGd5g5eCYPpCPDUy0FJXbYsaT5zYxjFOckS53SQDE3pWkVoWpHXVb3BrYcEN4Twa55B5cA=="],

    "wrap-ansi": ["wrap-ansi@8.1.0", "", { "dependencies": { "ansi-styles": "^6.1.0", "string-width": "^5.0.1", "strip-ansi": "^7.0.1" } }, "sha512-si7QWI6zUMq56bESFvagtmzMdGOtoxfR+Sez11Mobfc7tm+VkUckk9bW2UeffTGVUbOksxmSw0AA2gs8g71NCQ=="],

    "wrap-ansi-cjs": ["wrap-ansi@7.0.0", "", { "dependencies": { "ansi-styles": "^4.0.0", "string-width": "^4.1.0", "strip-ansi": "^6.0.0" } }, "sha512-YVGIj2kamLSTxw6NsZjoBxfSwsn0ycdesmc4p+Q21c5zPuZ1pl+NfxVdxPtdHvmNVOQ6XSYG4AUtyt/Fi7D16Q=="],

    "ws": ["ws@8.19.0", "", { "peerDependencies": { "bufferutil": "^4.0.1", "utf-8-validate": ">=5.0.2" }, "optionalPeers": ["bufferutil", "utf-8-validate"] }, "sha512-blAT2mjOEIi0ZzruJfIhb3nps74PRWTCz1IjglWEEpQl5XS/UNama6u2/rjFkDDouqr4L67ry+1aGIALViWjDg=="],

    "xml-name-validator": ["xml-name-validator@4.0.0", "", {}, "sha512-ICP2e+jsHvAj2E2lIHxa5tjXRlKDJo4IdvPvCXbXQGdzSfmSpNVyIKMvoZHjDY9DP0zV17iI85o90vRFXNccRw=="],

    "xmlchars": ["xmlchars@2.2.0", "", {}, "sha512-JZnDKK8B0RCDw84FNdDAIpZK+JuJw+s7Lz8nksI7SIuU3UXJJslUthsi+uWBUYOwPFwW7W7PRLRfUKpxjtjFCw=="],

    "yaml": ["yaml@2.6.0", "", { "bin": "bin.mjs" }, "sha512-a6ae//JvKDEra2kdi1qzCyrJW/WZCgFi8ydDV+eXExl95t+5R+ijnqHJbz9tmMh8FUjx3iv2fCQ4dclAQlO2UQ=="],

    "yocto-queue": ["yocto-queue@0.1.0", "", {}, "sha512-rVksvsnNCdJ/ohGc6xgPwyN8eheCxsiLM8mxuE/t/mOVqJewPuO1miLpTHQiRgTKCLexL4MeAFVagts7HmNZ2Q=="],

    "zod": ["zod@3.25.76", "", {}, "sha512-gzUt/qt81nXsFGKIFcC3YnfEAx5NkunCfnDlvuBSSFS02bcXu4Lmea0AFIUwbLWxWPx3d9p8S5QoaujKcNQxcQ=="],

    "zwitch": ["zwitch@2.0.4", "", {}, "sha512-bXE4cR/kVZhKZX/RjPEflHaKVhUVl85noU3v6b8apfQEc1x4A+zBxjZ4lN8LqGd6WZ3dl98pY4o717VFmoPp+A=="],

    "@eslint-community/eslint-utils/eslint-visitor-keys": ["eslint-visitor-keys@3.4.3", "", {}, "sha512-wpc+LXeiyiisxPlEkUzU6svyS1frIO3Mgxj1fdy7Pm8Ygzguax2N3Fa/D/ag1WqbOprdI+uY6wMUl8/a2G+iag=="],

    "@eslint/eslintrc/globals": ["globals@14.0.0", "", {}, "sha512-oahGvuMGQlPw/ivIYBjVSrWAfWLBeku5tpPE2fOPLi+WHffIWbuh2tCjhyQhTBPMf5E9jDEH4FOmTYgYwbKwtQ=="],

    "@humanfs/node/@humanwhocodes/retry": ["@humanwhocodes/retry@0.3.1", "", {}, "sha512-JBxkERygn7Bv/GbN5Rv8Ul6LVknS+5Bp6RgDC/O8gEBU/yeH5Ui5C/OlWrTb6qct7LjjfT6Re2NxB0ln0yYybA=="],

    "@typescript-eslint/eslint-plugin/ignore": ["ignore@7.0.5", "", {}, "sha512-Hs59xBNfUIunMFgWAbGX5cq6893IbWg4KnrjbYwX3tx0ztorVgTDA6B2sxf8ejHJ4wz8BqGUMYlnzNBer5NvGg=="],

    "@typescript-eslint/typescript-estree/minimatch": ["minimatch@9.0.5", "", { "dependencies": { "brace-expansion": "^2.0.1" } }, "sha512-G6T0ZX48xgozx7587koeX9Ys2NYy6Gmv//P89sEte9V9whIapMNF4idKxnW2QtCcLiTWlb/wfCabAtAFWhhBow=="],

    "anymatch/picomatch": ["picomatch@2.3.1", "", {}, "sha512-JU3teHTNjmE2VCGFzuY8EXzCDVwEqB2a8fsIvwaStHhAWJEeVd1o1QD80CU6+ZdEXXSLbSsuLwJjkCBWqRQUVA=="],

    "chokidar/glob-parent": ["glob-parent@5.1.2", "", { "dependencies": { "is-glob": "^4.0.1" } }, "sha512-AOIgSQCepiJYwP3ARnGx+5VnTu2HBYdzbGP45eLw1vr3zB3vZLeyed1sC9hnbcOc9/SrMyM5RPQrkGz4aS9Zow=="],

    "cssstyle/cssom": ["cssom@0.3.8", "", {}, "sha512-b0tGHbfegbhPJpxpiBPU2sCkigAqtM9O121le6bbOlgyV+NyGyCmVfJ6QW9eRjz8CpNfWEOYBIMIGRYkLwsIYg=="],

    "fast-glob/glob-parent": ["glob-parent@5.1.2", "", { "dependencies": { "is-glob": "^4.0.1" } }, "sha512-AOIgSQCepiJYwP3ARnGx+5VnTu2HBYdzbGP45eLw1vr3zB3vZLeyed1sC9hnbcOc9/SrMyM5RPQrkGz4aS9Zow=="],

    "glob/minimatch": ["minimatch@9.0.5", "", { "dependencies": { "brace-expansion": "^2.0.1" } }, "sha512-G6T0ZX48xgozx7587koeX9Ys2NYy6Gmv//P89sEte9V9whIapMNF4idKxnW2QtCcLiTWlb/wfCabAtAFWhhBow=="],

    "micromatch/picomatch": ["picomatch@2.3.1", "", {}, "sha512-JU3teHTNjmE2VCGFzuY8EXzCDVwEqB2a8fsIvwaStHhAWJEeVd1o1QD80CU6+ZdEXXSLbSsuLwJjkCBWqRQUVA=="],

    "parse-entities/@types/unist": ["@types/unist@2.0.11", "", {}, "sha512-CmBKiL6NNo/OqgmMn95Fk9Whlp2mtvIv+KNpQKN2F4SjvrEesubTRWGYSg+BnWZOnlCaSTU1sMpsBOzgbYhnsA=="],

    "postcss-nested/postcss-selector-parser": ["postcss-selector-parser@6.1.2", "", { "dependencies": { "cssesc": "^3.0.0", "util-deprecate": "^1.0.2" } }, "sha512-Q8qQfPiZ+THO/3ZrOrO0cJJKfpYCagtMUkXbnEfmgUjwXg6z/WBeOyS9APBBPCTSiDV+s4SwQGu8yFsiMRIudg=="],

    "prop-types/react-is": ["react-is@16.13.1", "", {}, "sha512-24e6ynE2H+OKt4kqsOvNd8kBpV65zoxbA4BVsEOB3ARVWQki/DHzaUoC5KuON/BiccDaCCTZBuOcfZs70kR8bQ=="],

    "readdirp/picomatch": ["picomatch@2.3.1", "", {}, "sha512-JU3teHTNjmE2VCGFzuY8EXzCDVwEqB2a8fsIvwaStHhAWJEeVd1o1QD80CU6+ZdEXXSLbSsuLwJjkCBWqRQUVA=="],

    "string-width-cjs/emoji-regex": ["emoji-regex@8.0.0", "", {}, "sha512-MSjYzcWNOA0ewAHpz0MxpYFvwg6yjy1NG3xteoqz644VCo/RPgnr1/GGt+ic3iJTzQ8Eu3TdM14SawnVUmGE6A=="],

    "string-width-cjs/strip-ansi": ["strip-ansi@6.0.1", "", { "dependencies": { "ansi-regex": "^5.0.1" } }, "sha512-Y38VPSHcqkFrCpFnQ9vuSXmquuv5oXOKpGeT6aGrr3o3Gc9AlVa6JBfUSOCnbxGGZF+/0ooI7KrPuUSztUdU5A=="],

    "strip-ansi-cjs/ansi-regex": ["ansi-regex@5.0.1", "", {}, "sha512-quJQXlTSUGL2LH9SUXo8VwsY4soanhgo6LNSm84E1LBcE8s3O0wpdiRzyR9z/ZZJMlMWv37qOOb9pdJlMUEKFQ=="],

    "strip-literal/js-tokens": ["js-tokens@9.0.1", "", {}, "sha512-mxa9E9ITFOt0ban3j6L5MpjwegGz6lBQmM1IJkWeBZGcMxto50+eWdjC/52xDbS2vy0k7vIMK0Fe2wfL9OQSpQ=="],

    "tailwindcss/postcss-selector-parser": ["postcss-selector-parser@6.1.2", "", { "dependencies": { "cssesc": "^3.0.0", "util-deprecate": "^1.0.2" } }, "sha512-Q8qQfPiZ+THO/3ZrOrO0cJJKfpYCagtMUkXbnEfmgUjwXg6z/WBeOyS9APBBPCTSiDV+s4SwQGu8yFsiMRIudg=="],

    "tinyglobby/picomatch": ["picomatch@4.0.3", "", {}, "sha512-5gTmgEY/sqK6gFXLIsQNH19lWb4ebPDLA4SdLP7dsWkIXHWlG66oPuVvXSGFPppYZz8ZDZq0dYYrbHfBCVUb1Q=="],

    "vite/esbuild": ["esbuild@0.21.5", "", { "optionalDependencies": { "@esbuild/aix-ppc64": "0.21.5", "@esbuild/android-arm": "0.21.5", "@esbuild/android-arm64": "0.21.5", "@esbuild/android-x64": "0.21.5", "@esbuild/darwin-arm64": "0.21.5", "@esbuild/darwin-x64": "0.21.5", "@esbuild/freebsd-arm64": "0.21.5", "@esbuild/freebsd-x64": "0.21.5", "@esbuild/linux-arm": "0.21.5", "@esbuild/linux-arm64": "0.21.5", "@esbuild/linux-ia32": "0.21.5", "@esbuild/linux-loong64": "0.21.5", "@esbuild/linux-mips64el": "0.21.5", "@esbuild/linux-ppc64": "0.21.5", "@esbuild/linux-riscv64": "0.21.5", "@esbuild/linux-s390x": "0.21.5", "@esbuild/linux-x64": "0.21.5", "@esbuild/netbsd-x64": "0.21.5", "@esbuild/openbsd-x64": "0.21.5", "@esbuild/sunos-x64": "0.21.5", "@esbuild/win32-arm64": "0.21.5", "@esbuild/win32-ia32": "0.21.5", "@esbuild/win32-x64": "0.21.5" }, "bin": "bin/esbuild" }, "sha512-mg3OPMV4hXywwpoDxu3Qda5xCKQi+vCTZq8S9J/EpkhB2HzKXq4SNFZE3+NK93JYxc8VMSep+lOUSC/RVKaBqw=="],

    "wrap-ansi/ansi-styles": ["ansi-styles@6.2.1", "", {}, "sha512-bN798gFfQX+viw3R7yrGWRqnrN2oRkEkUjjl4JNn4E8GxxbjtG3FbrEIIY3l8/hrwUwIeCZvi4QuOTP4MErVug=="],

    "wrap-ansi-cjs/string-width": ["string-width@4.2.3", "", { "dependencies": { "emoji-regex": "^8.0.0", "is-fullwidth-code-point": "^3.0.0", "strip-ansi": "^6.0.1" } }, "sha512-wKyQRQpjJ0sIp62ErSZdGsjMJWsap5oRNihHhu6G7JVO/9jIB6UyevL+tXuOqrng8j/cxKTWyWUwvSTriiZz/g=="],

    "wrap-ansi-cjs/strip-ansi": ["strip-ansi@6.0.1", "", { "dependencies": { "ansi-regex": "^5.0.1" } }, "sha512-Y38VPSHcqkFrCpFnQ9vuSXmquuv5oXOKpGeT6aGrr3o3Gc9AlVa6JBfUSOCnbxGGZF+/0ooI7KrPuUSztUdU5A=="],

    "@typescript-eslint/typescript-estree/minimatch/brace-expansion": ["brace-expansion@2.0.2", "", { "dependencies": { "balanced-match": "^1.0.0" } }, "sha512-Jt0vHyM+jmUBqojB7E1NIYadt0vI0Qxjxd2TErW94wDz+E2LAm5vKMXXwg6ZZBTHPuUlDgQHKXvjGBdfcF1ZDQ=="],

    "glob/minimatch/brace-expansion": ["brace-expansion@2.0.2", "", { "dependencies": { "balanced-match": "^1.0.0" } }, "sha512-Jt0vHyM+jmUBqojB7E1NIYadt0vI0Qxjxd2TErW94wDz+E2LAm5vKMXXwg6ZZBTHPuUlDgQHKXvjGBdfcF1ZDQ=="],

    "string-width-cjs/strip-ansi/ansi-regex": ["ansi-regex@5.0.1", "", {}, "sha512-quJQXlTSUGL2LH9SUXo8VwsY4soanhgo6LNSm84E1LBcE8s3O0wpdiRzyR9z/ZZJMlMWv37qOOb9pdJlMUEKFQ=="],

    "vite/esbuild/@esbuild/aix-ppc64": ["@esbuild/aix-ppc64@0.21.5", "", { "os": "aix", "cpu": "ppc64" }, "sha512-1SDgH6ZSPTlggy1yI6+Dbkiz8xzpHJEVAlF/AM1tHPLsf5STom9rwtjE4hKAF20FfXXNTFqEYXyJNWh1GiZedQ=="],

    "vite/esbuild/@esbuild/android-arm": ["@esbuild/android-arm@0.21.5", "", { "os": "android", "cpu": "arm" }, "sha512-vCPvzSjpPHEi1siZdlvAlsPxXl7WbOVUBBAowWug4rJHb68Ox8KualB+1ocNvT5fjv6wpkX6o/iEpbDrf68zcg=="],

    "vite/esbuild/@esbuild/android-arm64": ["@esbuild/android-arm64@0.21.5", "", { "os": "android", "cpu": "arm64" }, "sha512-c0uX9VAUBQ7dTDCjq+wdyGLowMdtR/GoC2U5IYk/7D1H1JYC0qseD7+11iMP2mRLN9RcCMRcjC4YMclCzGwS/A=="],

    "vite/esbuild/@esbuild/android-x64": ["@esbuild/android-x64@0.21.5", "", { "os": "android", "cpu": "x64" }, "sha512-D7aPRUUNHRBwHxzxRvp856rjUHRFW1SdQATKXH2hqA0kAZb1hKmi02OpYRacl0TxIGz/ZmXWlbZgjwWYaCakTA=="],

    "vite/esbuild/@esbuild/darwin-arm64": ["@esbuild/darwin-arm64@0.21.5", "", { "os": "darwin", "cpu": "arm64" }, "sha512-DwqXqZyuk5AiWWf3UfLiRDJ5EDd49zg6O9wclZ7kUMv2WRFr4HKjXp/5t8JZ11QbQfUS6/cRCKGwYhtNAY88kQ=="],

    "vite/esbuild/@esbuild/darwin-x64": ["@esbuild/darwin-x64@0.21.5", "", { "os": "darwin", "cpu": "x64" }, "sha512-se/JjF8NlmKVG4kNIuyWMV/22ZaerB+qaSi5MdrXtd6R08kvs2qCN4C09miupktDitvh8jRFflwGFBQcxZRjbw=="],

    "vite/esbuild/@esbuild/freebsd-arm64": ["@esbuild/freebsd-arm64@0.21.5", "", { "os": "freebsd", "cpu": "arm64" }, "sha512-5JcRxxRDUJLX8JXp/wcBCy3pENnCgBR9bN6JsY4OmhfUtIHe3ZW0mawA7+RDAcMLrMIZaf03NlQiX9DGyB8h4g=="],

    "vite/esbuild/@esbuild/freebsd-x64": ["@esbuild/freebsd-x64@0.21.5", "", { "os": "freebsd", "cpu": "x64" }, "sha512-J95kNBj1zkbMXtHVH29bBriQygMXqoVQOQYA+ISs0/2l3T9/kj42ow2mpqerRBxDJnmkUDCaQT/dfNXWX/ZZCQ=="],

    "vite/esbuild/@esbuild/linux-arm": ["@esbuild/linux-arm@0.21.5", "", { "os": "linux", "cpu": "arm" }, "sha512-bPb5AHZtbeNGjCKVZ9UGqGwo8EUu4cLq68E95A53KlxAPRmUyYv2D6F0uUI65XisGOL1hBP5mTronbgo+0bFcA=="],

    "vite/esbuild/@esbuild/linux-arm64": ["@esbuild/linux-arm64@0.21.5", "", { "os": "linux", "cpu": "arm64" }, "sha512-ibKvmyYzKsBeX8d8I7MH/TMfWDXBF3db4qM6sy+7re0YXya+K1cem3on9XgdT2EQGMu4hQyZhan7TeQ8XkGp4Q=="],

    "vite/esbuild/@esbuild/linux-ia32": ["@esbuild/linux-ia32@0.21.5", "", { "os": "linux", "cpu": "ia32" }, "sha512-YvjXDqLRqPDl2dvRODYmmhz4rPeVKYvppfGYKSNGdyZkA01046pLWyRKKI3ax8fbJoK5QbxblURkwK/MWY18Tg=="],

    "vite/esbuild/@esbuild/linux-loong64": ["@esbuild/linux-loong64@0.21.5", "", { "os": "linux", "cpu": "none" }, "sha512-uHf1BmMG8qEvzdrzAqg2SIG/02+4/DHB6a9Kbya0XDvwDEKCoC8ZRWI5JJvNdUjtciBGFQ5PuBlpEOXQj+JQSg=="],

    "vite/esbuild/@esbuild/linux-mips64el": ["@esbuild/linux-mips64el@0.21.5", "", { "os": "linux", "cpu": "none" }, "sha512-IajOmO+KJK23bj52dFSNCMsz1QP1DqM6cwLUv3W1QwyxkyIWecfafnI555fvSGqEKwjMXVLokcV5ygHW5b3Jbg=="],

    "vite/esbuild/@esbuild/linux-ppc64": ["@esbuild/linux-ppc64@0.21.5", "", { "os": "linux", "cpu": "ppc64" }, "sha512-1hHV/Z4OEfMwpLO8rp7CvlhBDnjsC3CttJXIhBi+5Aj5r+MBvy4egg7wCbe//hSsT+RvDAG7s81tAvpL2XAE4w=="],

    "vite/esbuild/@esbuild/linux-riscv64": ["@esbuild/linux-riscv64@0.21.5", "", { "os": "linux", "cpu": "none" }, "sha512-2HdXDMd9GMgTGrPWnJzP2ALSokE/0O5HhTUvWIbD3YdjME8JwvSCnNGBnTThKGEB91OZhzrJ4qIIxk/SBmyDDA=="],

    "vite/esbuild/@esbuild/linux-s390x": ["@esbuild/linux-s390x@0.21.5", "", { "os": "linux", "cpu": "s390x" }, "sha512-zus5sxzqBJD3eXxwvjN1yQkRepANgxE9lgOW2qLnmr8ikMTphkjgXu1HR01K4FJg8h1kEEDAqDcZQtbrRnB41A=="],

    "vite/esbuild/@esbuild/linux-x64": ["@esbuild/linux-x64@0.21.5", "", { "os": "linux", "cpu": "x64" }, "sha512-1rYdTpyv03iycF1+BhzrzQJCdOuAOtaqHTWJZCWvijKD2N5Xu0TtVC8/+1faWqcP9iBCWOmjmhoH94dH82BxPQ=="],

    "vite/esbuild/@esbuild/netbsd-x64": ["@esbuild/netbsd-x64@0.21.5", "", { "os": "none", "cpu": "x64" }, "sha512-Woi2MXzXjMULccIwMnLciyZH4nCIMpWQAs049KEeMvOcNADVxo0UBIQPfSmxB3CWKedngg7sWZdLvLczpe0tLg=="],

    "vite/esbuild/@esbuild/openbsd-x64": ["@esbuild/openbsd-x64@0.21.5", "", { "os": "openbsd", "cpu": "x64" }, "sha512-HLNNw99xsvx12lFBUwoT8EVCsSvRNDVxNpjZ7bPn947b8gJPzeHWyNVhFsaerc0n3TsbOINvRP2byTZ5LKezow=="],

    "vite/esbuild/@esbuild/sunos-x64": ["@esbuild/sunos-x64@0.21.5", "", { "os": "sunos", "cpu": "x64" }, "sha512-6+gjmFpfy0BHU5Tpptkuh8+uw3mnrvgs+dSPQXQOv3ekbordwnzTVEb4qnIvQcYXq6gzkyTnoZ9dZG+D4garKg=="],

    "vite/esbuild/@esbuild/win32-arm64": ["@esbuild/win32-arm64@0.21.5", "", { "os": "win32", "cpu": "arm64" }, "sha512-Z0gOTd75VvXqyq7nsl93zwahcTROgqvuAcYDUr+vOv8uHhNSKROyU961kgtCD1e95IqPKSQKH7tBTslnS3tA8A=="],

    "vite/esbuild/@esbuild/win32-ia32": ["@esbuild/win32-ia32@0.21.5", "", { "os": "win32", "cpu": "ia32" }, "sha512-SWXFF1CL2RVNMaVs+BBClwtfZSvDgtL//G/smwAc5oVK/UPu2Gu9tIaRgFmYFFKrmg3SyAjSrElf0TiJ1v8fYA=="],

    "vite/esbuild/@esbuild/win32-x64": ["@esbuild/win32-x64@0.21.5", "", { "os": "win32", "cpu": "x64" }, "sha512-tQd/1efJuzPC6rCFwEvLtci/xNFcTZknmXs98FYDfGE4wP9ClFV98nyKrzJKVPMhdDnjzLhdUyMX4PsQAPjwIw=="],

    "wrap-ansi-cjs/string-width/emoji-regex": ["emoji-regex@8.0.0", "", {}, "sha512-MSjYzcWNOA0ewAHpz0MxpYFvwg6yjy1NG3xteoqz644VCo/RPgnr1/GGt+ic3iJTzQ8Eu3TdM14SawnVUmGE6A=="],

    "wrap-ansi-cjs/strip-ansi/ansi-regex": ["ansi-regex@5.0.1", "", {}, "sha512-quJQXlTSUGL2LH9SUXo8VwsY4soanhgo6LNSm84E1LBcE8s3O0wpdiRzyR9z/ZZJMlMWv37qOOb9pdJlMUEKFQ=="],
  }
}
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "src/index.css",
    "baseColor": "slate",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  }
}
import js from "@eslint/js";
import globals from "globals";
import reactHooks from "eslint-plugin-react-hooks";
import reactRefresh from "eslint-plugin-react-refresh";
import tseslint from "typescript-eslint";

export default tseslint.config(
  { ignores: ["dist"] },
  {
    extends: [js.configs.recommended, ...tseslint.configs.recommended],
    files: ["**/*.{ts,tsx}"],
    languageOptions: {
      ecmaVersion: 2020,
      globals: globals.browser,
    },
    plugins: {
      "react-hooks": reactHooks,
      "react-refresh": reactRefresh,
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      "react-refresh/only-export-components": ["warn", { allowConstantExport: true }],
      "@typescript-eslint/no-unused-vars": "off",
    },
  },
);
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <!-- TODO: Set the document title to the name of your application -->
    <title>Lovable App</title>
    <meta name="description" content="Lovable Generated Project">
    <meta name="author" content="Lovable" />

    <!-- TODO: Update og:title to match your application name -->
    
    
    <meta property="og:type" content="website" />
    <meta property="og:image" content="https://pub-bb2e103a32db4e198524a2e9ed8f35b4.r2.dev/e627a53a-aaa4-469e-8fd8-7f7d74eaf4c3/id-preview-f8d00635--574f758b-d4e6-4715-876c-2000ebad9657.lovable.app-1772730904294.png">

    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:site" content="@Lovable" />
    <meta name="twitter:image" content="https://pub-bb2e103a32db4e198524a2e9ed8f35b4.r2.dev/e627a53a-aaa4-469e-8fd8-7f7d74eaf4c3/id-preview-f8d00635--574f758b-d4e6-4715-876c-2000ebad9657.lovable.app-1772730904294.png">
    <meta property="og:title" content="Lovable App">
  <meta name="twitter:title" content="Lovable App">
  <meta property="og:description" content="Lovable Generated Project">
  <meta name="twitter:description" content="Lovable Generated Project">
</head>

  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
{
  "name": "vite_react_shadcn_ts",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "build:dev": "vite build --mode development",
    "lint": "eslint .",
    "preview": "vite preview",
    "test": "vitest run",
    "test:watch": "vitest"
  },
  "dependencies": {
    "@hookform/resolvers": "^3.10.0",
    "@radix-ui/react-accordion": "^1.2.11",
    "@radix-ui/react-alert-dialog": "^1.1.14",
    "@radix-ui/react-aspect-ratio": "^1.1.7",
    "@radix-ui/react-avatar": "^1.1.10",
    "@radix-ui/react-checkbox": "^1.3.2",
    "@radix-ui/react-collapsible": "^1.1.11",
    "@radix-ui/react-context-menu": "^2.2.15",
    "@radix-ui/react-dialog": "^1.1.14",
    "@radix-ui/react-dropdown-menu": "^2.1.15",
    "@radix-ui/react-hover-card": "^1.1.14",
    "@radix-ui/react-label": "^2.1.7",
    "@radix-ui/react-menubar": "^1.1.15",
    "@radix-ui/react-navigation-menu": "^1.2.13",
    "@radix-ui/react-popover": "^1.1.14",
    "@radix-ui/react-progress": "^1.1.7",
    "@radix-ui/react-radio-group": "^1.3.7",
    "@radix-ui/react-scroll-area": "^1.2.9",
    "@radix-ui/react-select": "^2.2.5",
    "@radix-ui/react-separator": "^1.1.7",
    "@radix-ui/react-slider": "^1.3.5",
    "@radix-ui/react-slot": "^1.2.3",
    "@radix-ui/react-switch": "^1.2.5",
    "@radix-ui/react-tabs": "^1.1.12",
    "@radix-ui/react-toast": "^1.2.14",
    "@radix-ui/react-toggle": "^1.1.9",
    "@radix-ui/react-toggle-group": "^1.1.10",
    "@radix-ui/react-tooltip": "^1.2.7",
    "@supabase/supabase-js": "^2.98.0",
    "@tanstack/react-query": "^5.83.0",
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "cmdk": "^1.1.1",
    "date-fns": "^3.6.0",
    "embla-carousel-react": "^8.6.0",
    "framer-motion": "^12.35.0",
    "input-otp": "^1.4.2",
    "lucide-react": "^0.462.0",
    "next-themes": "^0.3.0",
    "react": "^18.3.1",
    "react-day-picker": "^8.10.1",
    "react-dom": "^18.3.1",
    "react-hook-form": "^7.61.1",
    "react-markdown": "^10.1.0",
    "react-resizable-panels": "^2.1.9",
    "react-router-dom": "^6.30.1",
    "recharts": "^2.15.4",
    "sonner": "^1.7.4",
    "tailwind-merge": "^2.6.0",
    "tailwindcss-animate": "^1.0.7",
    "vaul": "^0.9.9",
    "zod": "^3.25.76"
  },
  "devDependencies": {
    "@eslint/js": "^9.32.0",
    "@tailwindcss/typography": "^0.5.16",
    "@testing-library/jest-dom": "^6.6.0",
    "@testing-library/react": "^16.0.0",
    "@types/node": "^22.16.5",
    "@types/react": "^18.3.23",
    "@types/react-dom": "^18.3.7",
    "@vitejs/plugin-react-swc": "^3.11.0",
    "autoprefixer": "^10.4.21",
    "eslint": "^9.32.0",
    "eslint-plugin-react-hooks": "^5.2.0",
    "eslint-plugin-react-refresh": "^0.4.20",
    "globals": "^15.15.0",
    "jsdom": "^20.0.3",
    "lovable-tagger": "^1.1.13",
    "postcss": "^8.5.6",
    "tailwindcss": "^3.4.17",
    "typescript": "^5.8.3",
    "typescript-eslint": "^8.38.0",
    "vite": "^5.4.19",
    "vitest": "^3.2.4"
  }
}
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
# Welcome to your Lovable project

## Project info

**URL**: https://lovable.dev/projects/REPLACE_WITH_PROJECT_ID

## How can I edit this code?

There are several ways of editing your application.

**Use Lovable**

Simply visit the [Lovable Project](https://lovable.dev/projects/REPLACE_WITH_PROJECT_ID) and start prompting.

Changes made via Lovable will be committed automatically to this repo.

**Use your preferred IDE**

If you want to work locally using your own IDE, you can clone this repo and push changes. Pushed changes will also be reflected in Lovable.

The only requirement is having Node.js & npm installed - [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating)

Follow these steps:

```sh
# Step 1: Clone the repository using the project's Git URL.
git clone <YOUR_GIT_URL>

# Step 2: Navigate to the project directory.
cd <YOUR_PROJECT_NAME>

# Step 3: Install the necessary dependencies.
npm i

# Step 4: Start the development server with auto-reloading and an instant preview.
npm run dev
```

**Edit a file directly in GitHub**

- Navigate to the desired file(s).
- Click the "Edit" button (pencil icon) at the top right of the file view.
- Make your changes and commit the changes.

**Use GitHub Codespaces**

- Navigate to the main page of your repository.
- Click on the "Code" button (green button) near the top right.
- Select the "Codespaces" tab.
- Click on "New codespace" to launch a new Codespace environment.
- Edit files directly within the Codespace and commit and push your changes once you're done.

## What technologies are used for this project?

This project is built with:

- Vite
- TypeScript
- React
- shadcn-ui
- Tailwind CSS

## How can I deploy this project?

Simply open [Lovable](https://lovable.dev/projects/REPLACE_WITH_PROJECT_ID) and click on Share -> Publish.

## Can I connect a custom domain to my Lovable project?

Yes, you can!

To connect a domain, navigate to Project > Settings > Domains and click Connect Domain.

Read more here: [Setting up a custom domain](https://docs.lovable.dev/features/custom-domain#custom-domain)
import type { Config } from "tailwindcss";

export default {
  darkMode: ["class"],
  content: ["./pages/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}", "./app/**/*.{ts,tsx}", "./src/**/*.{ts,tsx}"],
  prefix: "",
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      fontFamily: {
        display: ["Space Grotesk", "sans-serif"],
        body: ["DM Sans", "sans-serif"],
      },
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
          dark: "hsl(var(--primary-dark))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
        sidebar: {
          DEFAULT: "hsl(var(--sidebar-background))",
          foreground: "hsl(var(--sidebar-foreground))",
          primary: "hsl(var(--sidebar-primary))",
          "primary-foreground": "hsl(var(--sidebar-primary-foreground))",
          accent: "hsl(var(--sidebar-accent))",
          "accent-foreground": "hsl(var(--sidebar-accent-foreground))",
          border: "hsl(var(--sidebar-border))",
          ring: "hsl(var(--sidebar-ring))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
        "float": {
          "0%, 100%": { transform: "translateY(0)" },
          "50%": { transform: "translateY(-8px)" },
        },
        "fade-in-up": {
          from: { opacity: "0", transform: "translateY(20px)" },
          to: { opacity: "1", transform: "translateY(0)" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
        "float": "float 3s ease-in-out infinite",
        "fade-in-up": "fade-in-up 0.5s ease-out forwards",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
} satisfies Config;
{
  "compilerOptions": {
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "lib": [
      "ES2020",
      "DOM",
      "DOM.Iterable"
    ],
    "module": "ESNext",
    "moduleDetection": "force",
    "moduleResolution": "bundler",
    "noEmit": true,
    "noFallthroughCasesInSwitch": false,
    "noImplicitAny": false,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "paths": {
      "@/*": [
        "./src/*"
      ]
    },
    "skipLibCheck": true,
    "strict": false,
    "target": "ES2020",
    "types": [
      "vitest/globals"
    ],
    "useDefineForClassFields": true
  },
  "include": [
    "src"
  ]
}{
  "compilerOptions": {
    "allowJs": true,
    "noImplicitAny": false,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "paths": {
      "@/*": [
        "./src/*"
      ]
    },
    "skipLibCheck": true,
    "strictNullChecks": false
  },
  "files": [],
  "references": [
    {
      "path": "./tsconfig.app.json"
    },
    {
      "path": "./tsconfig.node.json"
    }
  ]
}
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2023"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noEmit": true,

    /* Linting */
    "strict": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["vite.config.ts"]
}
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react-swc";
import path from "path";
import { componentTagger } from "lovable-tagger";

// https://vitejs.dev/config/
export default defineConfig(({ mode }) => ({
  server: {
    host: "::",
    port: 8080,
    hmr: {
      overlay: false,
    },
  },
  plugins: [react(), mode === "development" && componentTagger()].filter(Boolean),
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
}));
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react-swc";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./src/test/setup.ts"],
    include: ["src/**/*.{test,spec}.{ts,tsx}"],
  },
  resolve: {
    alias: { "@": path.resolve(__dirname, "./src") },
  },
});


