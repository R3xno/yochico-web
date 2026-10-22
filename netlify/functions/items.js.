const { MongoClient } = require("mongodb");

let client;
const CHANNEL = "HexaAuction"; // keep in sync with pokemon_adder.py
const TYPES = { 0: "legendary", 1: "non-legendary", 2: "shiny", 3: "tm" };

const nextMin = (c) => (c <= 50000 ? c + 500 : c <= 100000 ? c + 1000 : c + 2000);

const slug = (n) =>
  String(n || "")
    .toLowerCase()
    .trim()
    .replace(/é/g, "e")
    .replace(/♀/g, "-f")
    .replace(/♂/g, "-m")
    .replace(/[.'’:]/g, "")
    .replace(/\s+/g, "-")
    .replace(/[^a-z0-9\-]/g, "");

exports.handler = async () => {
  const headers = {
    "Content-Type": "application/json",
    "Cache-Control": "no-store",
    "Access-Control-Allow-Origin": "*",
  };
  try {
    if (!process.env.MONGO_URI) throw new Error("MONGO_URI not set");
    client = client || (await new MongoClient(process.env.MONGO_URI).connect());
    const docs = await client.db("auction_db").collection("kekz").find({}).toArray();
    const bot = (process.env.BOT_USERNAME || "HexaAuctionRobot").replace(/^@/, "");

    const items = docs
      .filter((p) => p.token)
      .map((p) => {
        const bids = p.bids || [];
        const base = p.base || 0;
        const current = bids.length ? bids[bids.length - 1] : base;
        let mid = p.msg_id;
        if (mid == null) {
          const n = parseInt(String(p.link || "").replace(/\/$/, "").split("/").pop(), 10);
          mid = isNaN(n) ? null : n;
        }
        return {
          token: p.token,
          name: p.pokemon_name || "Unknown",
          slug: slug(p.pokemon_name),
          nature: p.nature || "",
          level: p.level,
          type: TYPES[p.tipe ?? 1] || "non-legendary",
          boosted: p.boosted,
          info: p.info,
          base,
          current,
          bids: bids.length,
          next_min: nextMin(current),
          post: mid ? `https://t.me/${CHANNEL}/${mid}` : null,
          bid_url: `https://t.me/${bot}?start=${p.token}`,
        };
      });

    return { statusCode: 200, headers, body: JSON.stringify({ items }) };
  } catch (e) {
    console.error("items error:", e.message);
    client = undefined;
    return { statusCode: 500, headers, body: JSON.stringify({ items: [], error: "db" }) };
  }
};
