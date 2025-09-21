# How to Add Streaming in AutoFilter Bot Without Any Server or Hosting

You don’t need to host your streaming bot on any server. This way your AutoFilter runs smoothly without using too much CPU or memory.

### Steps to follow:

---

### 1. Add these bots to your database channel
* [@filestream_bot](https://t.me/filestream_bot)
* [@cl1to5bot](https://t.me/cl1to5bot)
* [@cl2to5bot](https://t.me/cl1to5bot)
* [@cl3to5bot](https://t.me/cl1to5bot)
* [@cl4to5bot](https://t.me/cl1to5bot)
* [@cl5to5bot](https://t.me/cl1to5bot)

Because of Telegram rate limits, we use multiple bots so streaming works smoothly.

---

### 2. Find your `/start` command handler

Usually it’s inside `plugins/commands.py`. (I’ll assume it’s `commands.py` here.)

---

### 3. Check your `requirements.txt`

See if `httpx` is already listed. If not, add this line:

```
httpx
```

---

### 4. Check your `commands.py`

Make sure you have:

```python
import httpx
```

If not, add it.

---

### 5. Add this function before your `/start` command

```python
async def get_stream_link(channel_id: int, message_id: int, fallback_link: str) -> str:
    bsse_url = "https://stream.codeltix.com"
    api_url = f"{bsse_url}/api/v1/hash/{channel_id}/{message_id}"
    
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(api_url)
        except httpx.RequestError as e:
            print(f"API request failed: {e}")
            return fallback_link

    if response.status_code != 200:
        print(f"API call failed: {response.status_code}, Error: {response.text}")
        return fallback_link

    try:
        data = response.json()
    except json.JSONDecodeError as e:
        print(f"Failed to decode JSON: {e}, Text: {response.text}")
        return fallback_link
    
    res_data = data.get("data")
    if not res_data:
        return fallback_link

    msg_id = res_data.get("message_id")
    ch_id = res_data.get("channel_id")
    hash_val = res_data.get("hash")

    if not all([msg_id, ch_id, hash_val]):
        return fallback_link

    return f"{bsse_url}/{ch_id}/{msg_id}?hash={hash_val}"
```

---

### 6. Find where the file is being sent in `/start`

It usually looks like this:

```python
@Client.on_message(filters.command("start") & filters.incoming)
async def start(client: Client, message):
    ...
    a = await client.send_cached_media(
        chat_id=message.from_user.id,
        file_id=file_id,
        caption=f_caption,
    )
```

---

### 7. Check for InlineKeyboard buttons

If there are already buttons, you’ll see something like this:

```python
btn = [[
    InlineKeyboardButton("Old Button", url="http://example.com"),
]]
...
a = await client.send_cached_media(
    chat_id=message.from_user.id,
    file_id=file_id,
    caption=f_caption,
    reply_markup=InlineKeyboardMarkup(btn)
)
```

---

### 8. If no buttons exist, add one for streaming

```python
btn = [[
    InlineKeyboardButton("Streaming Link", url=await get_stream_link(BIN_CHANNEL_ID, file_msg.id, "https://a.random.link")),
]]
...
a = await client.send_cached_media(
    chat_id=message.from_user.id,
    file_id=file_id,
    caption=f_caption,
    reply_markup=InlineKeyboardMarkup(btn)
)
```

---

### 9. If buttons already exist, just add a new one

```python
btn = [[
    InlineKeyboardButton("Old Button", url="http://example.com"),
    InlineKeyboardButton("Streaming Link", url=await get_stream_link(BIN_CHANNEL_ID, file_msg.id, "https://a.random.link")),
]]
...
a = await client.send_cached_media(
    chat_id=message.from_user.id,
    file_id=file_id,
    caption=f_caption,
    reply_markup=InlineKeyboardMarkup(btn)
)
```

---

### 10. Send the file to BIN\_CHANNEL first

Above your button code, add this:

```python
file_msg = await client.send_cached_media(
    chat_id=BIN_CHANNEL_ID,
    file_id=file.file_id,
    caption=f_caption,
)
btn = [[
    InlineKeyboardButton("Old Button", url="http://example.com"),
    InlineKeyboardButton("Streaming Link", url=await get_stream_link(BIN_CHANNEL_ID, file_msg.id, "https://a.random.link")),
]]
```

---

### 11. Final code example

```python
file_msg = await client.send_cached_media(
    chat_id=BIN_CHANNEL_ID,
    file_id=file.file_id,
    caption=f_caption,
)
btn = [[
    InlineKeyboardButton("Old Button", url="http://example.com"),
    InlineKeyboardButton("Streaming Link", url=await get_stream_link(BIN_CHANNEL_ID, file_msg.id, "https://a.random.link")),
]]
a = await client.send_cached_media(
    chat_id=message.from_user.id,
    file_id=file_id,
    caption=f_caption,
    reply_markup=InlineKeyboardMarkup(btn)
)
```

---

### 12. Add BIN channel ID in `info.py` or `var.py`

```python
BIN_CHANNEL_ID = 123456789
```

---

### 13. Import in `commands.py`

```python
import httpx
from info import BIN_CHANNEL_ID
```

---

✅ Done! Now your AutoFilter bot has streaming links without needing any extra server or hosting.

