# i made a typing game where you explode if you can't spell

so i accidentally built a typing game that's now a full 3D space combat sim

the pitch: enemy ships shoot missiles at you. each missile has a word on it. type the word → your ship fires back → missile explodes. miss it → you take damage. die → leaderboard of shame.

the accident part is that it kept growing. now it has:
- 7 game modes (including one where an AI narrates a story and the words from the story try to kill you??)
- multiplayer where both players get the same word sequence from a shared seed so it's actually fair
- a spaced repetition engine quietly making you better at the words you keep fumbling
- tournaments with brackets and match lobbies
- 274k words so good luck memorizing the pool

built it with three.js, react, fastify, supabase. the whole thing runs in a browser.

i'm unreasonably proud of the "last missile" mechanic — when there's one word left and it's about to hit you, it slows down 20%. stolen directly from tetris. feels so good.

anyway. typesh!t. come type or die.

#BuiltWithMeDo
