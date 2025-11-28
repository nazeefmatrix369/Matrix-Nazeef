export default {
  async fetch(request, env) {

    const url = new URL(request.url);

    // Health check برای Cloudflare
    if (url.pathname === "/healthz") {
      return new Response("OK");
    }

    // دریافت webhook از Telegram
    if (request.method === "POST") {
      let update = await request.json();
      let chat_id = update.message?.chat?.id;
      let text = update.message?.text;

      if (!chat_id || !text) {
        return new Response("NO_MESSAGE");
      }

      // اگر لینک فرستادند
      if (text.startsWith("http")) {
        await sendMessage(env, chat_id, "⏳ در حال پردازش لینک...");

        // استفاده از API آماده برای دانلود
        const apiUrl = `https://api.letsdown.io/download?url=${encodeURIComponent(text)}`;
        const res = await fetch(apiUrl);
        const data = await res.json();

        if (!data || !data.url) {
          await sendMessage(env, chat_id, "❌ لینک معتبر نیست!");
          return new Response("INVALID");
        }

        // ارسال فایل
        await sendDocument(env, chat_id, data.url);
        return new Response("DONE");
      }

      // اگر پیام معمولی بود
      await sendMessage(env, chat_id, "سلام! لینک را بفرستید تا دانلود شود. 🔥");
      return new Response("OK");
    }

    return new Response("MATRIX DOWNLOADER ACTIVE");
  }
};

// ارسال پیام ساده
async function sendMessage(env, chat_id, text) {
  return fetch(`https://api.telegram.org/bot${env.BOT_TOKEN}/sendMessage`, {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({ chat_id, text })
  });
}

// ارسال فایل دانلود‌شده
async function sendDocument(env, chat_id, file_url) {
  return fetch(`https://api.telegram.org/bot${env.BOT_TOKEN}/sendDocument`, {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({
      chat_id,
      document: file_url
    })
  });
}
