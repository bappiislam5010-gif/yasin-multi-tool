
async function send(env, method, data) {
  return fetch(
    `https://api.telegram.org/bot${env.BOT_TOKEN}/${method}`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data)
    }
  );
}

function menu() {
  return {
    inline_keyboard: [
      [
        { text: "📹 Video Downloader", callback_data: "download" }
      ],
      [
        { text: "📚 YouTube Free Courses", callback_data: "courses" }
      ],
      [
        { text: "🎙️ AI Voice Generator", callback_data: "voice" }
      ],
      [
        { text: "ℹ️ Help & Support", callback_data: "help" }
      ]
    ]
  };
}

export default {
  async fetch(request, env) {
    const url = new URL(request.url);

    // One-time webhook setup
    if (url.pathname === "/setup" && request.method === "GET") {
      if (!env.SETUP_KEY ||
          url.searchParams.get("key") !== env.SETUP_KEY) {
        return new Response("Unauthorized", { status: 401 });
      }

      const result = await send(env, "setWebhook", {
        url: url.origin + "/webhook",
        secret_token: env.WEBHOOK_SECRET
      });

      return new Response(await result.text());
    }

    if (url.pathname !== "/webhook") {
      return new Response("Yasin Multi Tool Bot is running!");
    }

    if (
      request.headers.get("X-Telegram-Bot-Api-Secret-Token")
      !== env.WEBHOOK_SECRET
    ) {
      return new Response("Unauthorized", { status: 401 });
    }

    const update = await request.json();

    if (update.callback_query) {
      const q = update.callback_query;
      const chatId = q.message.chat.id;

      await send(env, "answerCallbackQuery", {
        callback_query_id: q.id
      });

      let message = "";

      if (q.data === "download") {
        message =
          "📹 Video Downloader\n\n" +
          "এই ফিচারে অনুমোদিত ভিডিও লিংক থেকে " +
          "ডাউনলোডের ব্যবস্থা যুক্ত করা হবে।";
      }

      if (q.data === "courses") {
        await send(env, "sendMessage", {
          chat_id: chatId,
          text: "📚 YouTube Free Courses\n\nকোর্সের ক্যাটাগরি বেছে নাও:",
          reply_markup: {
            inline_keyboard: [
              [{ text: "🎬 YouTube Basics",
                 url: "https://www.youtube.com/results?search_query=YouTube+course+Bangla" }],
              [{ text: "✂️ Video Editing",
                 url: "https://www.youtube.com/results?search_query=Video+Editing+Bangla+Course" }],
              [{ text: "🎨 Thumbnail Design",
                 url: "https://www.youtube.com/results?search_query=Thumbnail+Design+Bangla" }],
              [{ text: "🔙 Main Menu", callback_data: "home" }]
            ]
          }
        });
        return new Response("OK");
      }

      if (q.data === "voice") {
        message =
          "🎙️ AI Voice Generator\n\n" +
          "AI Voice ফিচারটি পরের ধাপে যুক্ত হবে।";
      }

      if (q.data === "help") {
        message = "ℹ️ Yasin Multi Tool Bot\nসাহায্যের জন্য Admin-এর সঙ্গে যোগাযোগ করো।";
      }

      if (q.data === "home") {
        await send(env, "sendMessage", {
          chat_id: chatId,
          text: "🏠 Main Menu\nএকটি ফিচার নির্বাচন করো:",
          reply_markup: menu()
        });
        return new Response("OK");
      }

      await send(env, "sendMessage", {
        chat_id: chatId,
        text: message,
        reply_markup: menu()
      });

      return new Response("OK");
    }

    const msg = update.message;
    if (!msg || !msg.text) return new Response("OK");

    if (msg.text === "/start") {
      await send(env, "sendMessage", {
        chat_id: msg.chat.id,
        text:
          "🎉 Welcome to Yasin Multi Tool Bot!\n\n" +
          "একটি বটেই পাচ্ছো তিনটি সুবিধা।\n\n" +
          "👇 নিচ থেকে তোমার পছন্দের ফিচার নির্বাচন করো:",
        reply_markup: menu()
      });
    } else {
      await send(env, "sendMessage", {
        chat_id: msg.chat.id,
        text: "🏠 Main Menu থেকে একটি অপশন নির্বাচন করো।",
        reply_markup: menu()
      });
    }

    return new Response("OK");
  }
};
# yasin-multi-tool
