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
    if (user) {
      navigate(intent === "sell" ? "/sell" : "/shop");
    } else {
      navigate(`/auth?intent=${intent}`);
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
            <h1 className="mb-4 font-display text-4xl font-bold tracking-tight text-primary-foreground md:text-5xl lg:text-6xl">
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
            <h2 className="font-display text-2xl font-bold text-foreground">Najnovije knjige</h2>
            <p className="text-sm text-muted-foreground mt-1">Poslednje dodati udžbenici na platformi</p>
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
