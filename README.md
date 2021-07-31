### Hi there 👋

<!--
**Wendy-star/Wendy-star** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
--> Skip to content
fuxionkali
/
Selfbot-whatsapp
forked from Miyaa8/Selfbot-whatsapp
Code
Pull requests
Actions
Projects
Wiki
Security
Insights
Selfbot-whatsapp/index.js
@fuxionkali
fuxionkali Add files via upload
 4 contributors
1394 lines (1364 sloc)  67.2 KB
/** 
 * Join Group : https://chat.whatsapp.com/HzsrDmMZ1sFFlac2JNhccJ
 * Follow IG Lindow : https://instagram.com/lindoww.8
 * Oh ya, sorry kalo code nya berantakan, gatau juga padahal di github nya rapih :)
 */

const { 
  WAConnection,
  MessageType,
  Presence, 
  MessageOptions,
  Mimetype,
  WALocationMessage,
  WA_MESSAGE_STUB_TYPES,
  ReconnectMode,
  ProxyAgent,
  GroupSettingChange,
  ChatModification,
  waChatKey,
  WA_DEFAULT_EPHEMERAL,
  mentionedJid,
  processTime
} = require("@adiwajshing/baileys")
const moment = require("moment-timezone");
const FormData = require('form-data')
const imageToBase64 = require('image-to-base64');
const speed = require('performance-now');
const chalk = require('chalk');
const request = require('request');
const fs = require('fs');
const brainly = require('brainly-scraper');
const { exec } = require('child_process');
const ffmpeg = require('fluent-ffmpeg');
const axios = require('axios');

const { fetchJson } = require('./lib/fetcher');
const { ssstik } = require("./lib/tiktok.js")
const { Gempa } = require("./lib/gempa.js");
const { SearchKartun, Movie, Drama, Action, Adventure } = require("./lib/kartun.js")
const { herolist } = require("./lib/herolist.js")
const { herodetails } = require("./lib/herodetail.js")
const conn = require("./lib/connect")
const msg = require("./lib/message")
const wa = require("./lib/wa")
const Exif = require('./lib/exif');
const exif = new Exif();
const { recognize } = require('./lib/ocr');
const help = require("./lib/help")
const postBuffer = help.postBuffer
const getBuffer = help.getBuffer
const getRandom = help.getRandomExt
const postJson = help.postJson
const getJson = help.getJson
const config = JSON.parse(fs.readFileSync("./config.json"))
const owner = config.owner
const mods = config.mods
const public = config.public

// Database
const imagenye = JSON.parse(fs.readFileSync('./database/image.json'))

conn.connect()
const megayaa = conn.megayaa

const sleep = async (ms) => {
    return new Promise(resolve => setTimeout(resolve, ms));
}

fakeimage = fs.readFileSync(`./lib/image/foto2.jpg`)
fake = 'ᴛɪᴛᴏ sᴇʟғʙᴏᴛ'
prefix = 'z'
apikey = 'LindowApi' // Free Apikey!
hit_today = []

megayaa.on('CB:action,,call', async json => {
    const callerId = json[2][0][1].from;
    console.log("call dari "+ callerId)
        megayaa.sendMessage(callerId, "Auto block system, don't call please", MessageType.text)
        await sleep(4000)
        await megayaa.blockUser(callerId, "add") // Block user
})

megayaa.on('group-participants-update', async(chat) => {
    try {
        var member = chat.participants
        for (var x of member) {
            try {
                if (x == megayaa.user.jid) return
                var photo = await wa.getPictProfile(x)
                var username = await wa.getUserName(x) || "Guest"
                var from = chat.jid
                var group = await megayaa.groupMetadata(from)
                if (chat.action == 'add' && public) {
                     text = `${username}, Wecome to ${group.subject}`
                        wa.sendImage(from, image, text)
                }
                if (chat.action == 'remove' && public) {
                    text = `${username}, Sayonara 👋`
                    await wa.sendMessage(from, text)
                }
            } catch {
                continue
            }
        }
    } catch (e) {
        console.log(chalk.whiteBright("├"), chalk.keyword("aqua")("[  ERROR  ]"), chalk.keyword("red")(e))
    }
})

megayaa.on('chat-update', async(lin) => {
    try {
        if (!lin.hasNewMessage) return
        if (!lin.messages) return
        if (lin.key && lin.key.remoteJid == 'status@broadcast') return
        lin = lin.messages.all()[0]
        if (!lin.message) return
        const from = lin.key.remoteJid
        const type = Object.keys(lin.message)[0]
        const { text, extendedText, contact, location, liveLocation, image, video, sticker, document, audio, product } = MessageType
        const quoted = type == 'extendedTextMessage' && lin.message.extendedTextMessage.contextInfo != null ? lin.message.extendedTextMessage.contextInfo.quotedMessage || [] : []
        const typeQuoted = Object.keys(quoted)[0]
        const body = lin.message.conversation || lin.message[type].caption || lin.message[type].text || ""
        chats = (type === 'conversation') ? lin.message.conversation : (type === 'extendedTextMessage') ? lin.message.extendedTextMessage.text : ''
        budy = (type === 'conversation' && lin.message.conversation.startsWith(prefix)) ? lin.message.conversation : (type == 'imageMessage') && lin.message.imageMessage.caption.startsWith(prefix) ? lin.message.imageMessage.caption : (type == 'videoMessage') && lin.message.videoMessage.caption.startsWith(prefix) ? lin.message.videoMessage.caption : (type == 'extendedTextMessage') && lin.message.extendedTextMessage.text.startsWith(prefix) ? lin.message.extendedTextMessage.text : ''

        if (prefix != "") {
            if (!body.startsWith(prefix)) {
                cmd = false
                comm = ""
            } else {
                cmd = true
                comm = body.slice(1).trim().split(" ").shift().toLowerCase()
            }
        } else {
            cmd = false
            comm = body.trim().split(" ").shift().toLowerCase()
        }

        const reply = async(teks) => {
            await megayaa.sendMessage(from, teks, MessageType.text, { quoted: lin })
        }

        const command = comm
        hit_today.push(command)
        const args = body.trim().split(/ +/).slice(1)
        const isCmd = cmd
        const meNumber = megayaa.user.jid
        const botNumber = megayaa.user.jid.split("@")[0]
        const isGroup = from.endsWith('@g.us')
        const arg = chats.slice(command.length + 2, chats.length)
        const sender = lin.key.fromMe ? megayaa.user.jid : isGroup ? lin.participant : lin.key.remoteJid
        const senderNumber = sender.split("@")[0]
        const groupMetadata = isGroup ? await megayaa.groupMetadata(from) : ''
        const groupName = isGroup ? groupMetadata.subject : ''
        const groupMembers = isGroup ? groupMetadata.participants : ''
        const groupAdmins = isGroup ? await wa.getGroupAdmins(groupMembers) : []
        const isAdmin = groupAdmins.includes(sender) || false
        const botAdmin = groupAdmins.includes(megayaa.user.jid)
        const totalChat = megayaa.chats.all()
        const itsMe = senderNumber == botNumber
        const isOwner = senderNumber == owner || senderNumber == botNumber || mods.includes(senderNumber)
        
        const mentionByTag = type == "extendedTextMessage" && lin.message.extendedTextMessage.contextInfo != null ? lin.message.extendedTextMessage.contextInfo.mentionedJid : []
        const mentionByReply = type == "extendedTextMessage" && lin.message.extendedTextMessage.contextInfo != null ? lin.message.extendedTextMessage.contextInfo.participant || "" : ""
        const mention = typeof(mentionByTag) == 'string' ? [mentionByTag] : mentionByTag
        mention != undefined ? mention.push(mentionByReply) : []
        const mentionUser = mention != undefined ? mention.filter(n => n) : []
        const mentions = (teks, memberr, id) => {
	    (id == null || id == undefined || id == false) ? megayaa.sendMessage(from, teks.trim(), extendedText, {contextInfo: {"mentionedJid": memberr}}) : megayaa.sendMessage(from, teks.trim(), extendedText, {quoted: lin, contextInfo: {"mentionedJid": memberr}})
	}
	    	
        // Ucapan Waktu
        const hour_now = moment().format('HH')
        var ucapanWaktu = 'Pagi, semangat ya'
        if (hour_now >= '03' && hour_now <= '10') {
          ucapanWaktu = 'Selamat pagi Tito'
        } else if (hour_now >= '10' && hour_now <= '14') {
          ucapanWaktu = 'Selamat siang Tito'
        } else if (hour_now >= '14' && hour_now <= '17') {
          ucapanWaktu = 'Selamat sore Tito'
        } else if (hour_now >= '17' && hour_now <= '18') {
          ucapanWaktu = 'Petang'
        } else if (hour_now >= '18' && hour_now <= '23') {
          ucapanWaktu = 'Selamat malam Tito'
        } else {
          ucapanWaktu = 'Selamat Malam!'
        }

        const isImage = type == 'imageMessage'
        const isVideo = type == 'videoMessage'
        const isAudio = type == 'audioMessage'
        const isSticker = type == 'stickerMessage'
        const isContact = type == 'contactMessage'
        const isLocation = type == 'locationMessage'
        const isMedia = (type === 'imageMessage' || type === 'videoMessage')
        
        typeMessage = body.substr(0, 50).replace(/\n/g, '')
        if (isImage) typeMessage = "Image"
        else if (isVideo) typeMessage = "Video"
        else if (isAudio) typeMessage = "Audio"
        else if (isSticker) typeMessage = "Sticker"
        else if (isContact) typeMessage = "Contact"
        else if (isLocation) typeMessage = "Location"

        const isQuoted = type == 'extendedTextMessage'
        const isQuotedImage = isQuoted && typeQuoted == 'imageMessage'
        const isQuotedVideo = isQuoted && typeQuoted == 'videoMessage'
        const isQuotedAudio = isQuoted && typeQuoted == 'audioMessage'
        const isQuotedSticker = isQuoted && typeQuoted == 'stickerMessage'
        const isQuotedContact = isQuoted && typeQuoted == 'contactMessage'
        const isQuotedLocation = isQuoted && typeQuoted == 'locationMessage'

        if (!public) {
            mods.indexOf(botNumber) === -1 ? mods.push(botNumber) : false
            mods.indexOf(owner) === -1 ? mods.push(owner) : false
            if (!mods.includes(senderNumber)) return
            mods.slice(mods.indexOf(owner), 1)
        }
        
        if (!isGroup && isCmd) console.log(chalk.whiteBright("├"), chalk.keyword("aqua")("[ COMMAND ]"), chalk.whiteBright(typeMessage), chalk.greenBright("from"), chalk.keyword("yellow")(senderNumber))
        if (isGroup && isCmd) console.log(chalk.whiteBright("├"), chalk.keyword("aqua")("[ COMMAND ]"), chalk.whiteBright(typeMessage), chalk.greenBright("from"), chalk.keyword("yellow")(senderNumber), chalk.greenBright("in"), chalk.keyword("yellow")(groupName))
        
        switch (command) {
            case 'owner':
                await wa.sendContact(from, owner, "Your Name")
                break
            case 'help':
                textnya = `*${ucapanWaktu}*
*> for eval*
*Hit Today : ${hit_today.length}*
*───────────────────*
ʟɪsᴛ ᴍᴇɴᴜ
*───────────────────*
  *├❏* ${prefix}owner
  *├❏* ${prefix}public
  *├❏* ${prefix}self
  *├❏* ${prefix}setprefix
  *├❏* ${prefix}broadcast
  *├❏* ${prefix}setthumb
  *├❏* ${prefix}fakethumb
  *├❏* ${prefix}stats
  *├❏* ${prefix}block
  *├❏* ${prefix}unblock
  *├❏* ${prefix}leave
  *├❏* ${prefix}join
  *├❏* ${prefix}clearall
  *├❏* ${prefix}hidetag
  *├❏* ${prefix}imagetag
  *├❏* ${prefix}stickertag
  *├❏* ${prefix}promote
  *├❏* ${prefix}demote
  *├❏* ${prefix}admin
  *├❏* ${prefix}linkgc
  *├❏* ${prefix}group open/close
  *├❏* ${prefix}setnamegc
  *├❏* ${prefix}setdesc
  *├❏* ${prefix}bugimg
  *├❏* ${prefix}demoteall
  *├❏* ${prefix}toimg
  *├❏* ${prefix}ocr
  *├❏* ${prefix}shutdown
  *├❏* ${prefix}spam
  *├❏* ${prefix}add
  *├❏* ${prefix}kick
  *├❏* ${prefix}setpp
  *├❏* ${prefix}chat
  *├❏* ${prefix}tagall
  *├❏* ${prefix}toptt
  *├❏* ${prefix}fordward
  *├❏* ${prefix}fakereply
  *├❏* ${prefix}unreadall
  *├❏* ${prefix}readall
  *├❏* ${prefix}upstory
  *├❏* ${prefix}upstorypic
  *├❏* ${prefix}upstoryvid
  *├❏* ${prefix}unmute
  *├❏* ${prefix}mute
  *├❏* ${prefix}delthischat
  *├❏* ${prefix}archive
  *├❏* ${prefix}unarchiveall
  *├❏* ${prefix}pin
  *├❏* ${prefix}unpin
  *├❏* ${prefix}runtime
  *├❏* ${prefix}speed
  *├❏* ${prefix}sendkontak
  *├❏* ${prefix}term
  *├❏* ${prefix}setreply
  *├❏* ${prefix}setname
  *├❏* ${prefix}setbio
  *├❏* ${prefix}fdeface
  *├❏* ${prefix}getpic
  *├❏* ${prefix}getbio
  *├❏* ${prefix}sticker
  *├❏* ${prefix}swm
  *├❏* ${prefix}takestick
  *├❏* ${prefix}colong
  *├❏* ${prefix}brainly
  *├❏* ${prefix}ytsearch
  *├❏* ${prefix}igdl
  *├❏* ${prefix}scdl
  *├❏* ${prefix}ppcouple
  *├❏* ${prefix}asupan
  *├❏* ${prefix}randomaesthetic
  *├❏* ${prefix}quoteislam
  *├❏* ${prefix}kisahnabi
  *├❏* ${prefix}ayatkursi
  *├❏* ${prefix}time
  *├❏* ${prefix}herolist
  *├❏* ${prefix}herodetail
  *├❏* ${prefix}searchkartun
  *├❏* ${prefix}kartunmovie
  *├❏* ${prefix}kartundrama
  *├❏* ${prefix}kartunadventure
  *├❏* ${prefix}cuaca
  *├❏* ${prefix}gempa
  *├❏* ${prefix}gempaterkini
  *├❏* ${prefix}jadwaltv
  *├❏* ${prefix}tinyurl
  *├❏* ${prefix}ssweb
  *├❏* ${prefix}translate
  *├❏* ${prefix}noprefix
  *├❏* ${prefix}pinterest
  *├❏* ${prefix}dewabatch
  *├❏* ${prefix}wikipedia
  *├❏* ${prefix}kusonime
  *├❏* ${prefix}play
  *├❏* ${prefix}ytmp3
  *├❏* ${prefix}ytmp4
  *├❏* ${prefix}addimage
  *├❏* ${prefix}listimage
  *├❏* ${prefix}getimage
*───────────────────*
  *├>*  creator : Lindow
*───────────────────*
`
            wa.FakeTokoForwarded(from, textnya, fake)
                break
				case 'translate':
			const tr = await axios.get(`https://lindow-api.herokuapp.com/api/translate?kata=${arg}&apikey=LindowApi`, {method: 'get'})
				  trya = `${tr.data.result.text}`
			reply(trya)
				  break
case 'jadwaltv':                             
tvnow = await fetchJson(`https://h4ck3rs404-api.herokuapp.com/api/jadwaltv?channel=${args}&apikey=404Api`)
reply(tvnow.result)
break
                case 'time':
    tanggal = await fetchJson('https://time.siswadi.com/Asia/Jakarta')
		anu = await fetchJson(`https://api.pray.zone/v2/times/day.json?city=${args}&date=${tanggal.data.date}`)
					lirik = `*Time of ${args}*\n*Imsak :* ${anu.results.datetime[0].times.Imsak}\n*Sunrise :* ${anu.results.datetime[0].times.Sunrise}\n*Fajar :* ${anu.results.datetime[0].times.Fajr}\n*Dhuzur :* ${anu.results.datetime[0].times.Dhuhr}\n*Ashar :* ${anu.results.datetime[0].times.Asr}\n*Sunset :* ${anu.results.datetime[0].times.Sunset}\n*Maghrib :* ${anu.results.datetime[0].times.Maghrib}\n*Isya :* ${anu.results.datetime[0].times.Isha}\n*Midnight :* ${anu.results.datetime[0].times.Midnight}`
					reply(lirik)
					break
                case 'brainly':
					brien = args
					brainly(`${brien}`).then(res => {
					teks = '❉────────────────❉\n'
					for (let Y of res.data) {
						teks += `\n*「 _BRAINLY_ 」*\n\n*➸ Pertanyaan:* ${Y.pertanyaan}\n\n*➸ Jawaban:* ${Y.jawaban[0].text}\n = '❉────────────────❉\n`
					}
		reply(teks)
					console.log(res)
					})

					break
                case 'ssweb':
					  web = await getBuffer(`https://shot.screenshotapi.net/screenshot?&url=${args[0]}&width=1280&height=720&full_page=true&fresh=true&output=image&file_type=png&block_ads=true&block_tracking=true`)
					  megayaa.sendMessage(from, web, image, {quoted: lin})
					  break
            case 'tiktok':
                url = args.join(" ")
                result = await ssstik(url)
                console.log(result)
                buf = await getBuffer(`${result.videonowm}`)
                megayaa.sendMessage(from, buf, video, {mimetype: 'video/mp4', filename: `tiktok.mp4`, quoted: lin, caption: `${result.text}\n\nUrl music : ${result.music}`})
                break
            case 'ytmp3':
                yt = await axios.get(`https://lindow-python-api.herokuapp.com/api/yta?url=${body.slice(7)}`)
                var { ext, filesize, result, thumb, title } = yt.data
                foto = await getBuffer(thumb)
                if (Number(filesize.split(' MB')[0]) >= 30.00) return megayaa.sendMessage(from, foto, image, {caption: `Title : ${title}\n\nExt : ${ext}\n\nFilesize : ${filesize}\n\nLink : ${result}\n\nUkuran audio diatas 30 MB, Silakan gunakan link download manual`})
                cap = `Ytmp3 downloader\n\nTitle : ${title}\n\nExt : ${ext}\n\nFilesize : ${filesize}`
                megayaa.sendMessage(from, foto, image, {caption: cap})
                au = await getBuffer(result)
                megayaa.sendMessage(from, au, audio, {mimetype: 'audio/mp4', filename: `${title}.mp3`, quoted: lin})
                break
	case 'play':
	  lagu = args.join(" ")
	  get_result = await fetchJson(`https://docs-jojo.herokuapp.com/api/yt-play?q=${lagu}`)
	ini_txt = (`Title : ${get_result.title}\nChannel : ${get_result.channel}\nDuration : ${get_result.duration}\nViews : ${get_result.total_view}\nSize : ${get_result.filesize}\nUploaded : ${get_result.uploaded}\n\nWait few minutes...`)
	ini_buffer = await getBuffer(get_result.thumb)
	megayaa.sendMessage(from, ini_buffer, image, {quoted: lin, caption: ini_txt})
	get_audio = await getBuffer(get_result.link)
	megayaa.sendMessage(from, get_audio, audio, {mimetype: 'audio/mp4', filename: `${get_result.title}.mp3`, quoted: lin})
	break
            case 'ytmp4':
                yt = await axios.get(`https://lindow-python-api.herokuapp.com/api/ytv?url=${body.slice(7)}`)
                var { ext, filesize, resolution, result, thumb, title } = yt.data
                foto = await getBuffer(thumb)
                if (Number(filesize.split(' MB')[0]) >= 30.00) return megayaa.sendMessage(from, foto, image, {caption: `Title : ${title}\n\nExt : ${ext}\n\nFilesize : ${filesize}\n\nResolution: ${resolution}\n\nLink : ${result}\n\nUkuran video diatas 30 MB, Silakan gunakan link download manual`})
                cap = `Ytmp4 downloader\n\nTitle : ${title}\n\nExt : ${ext}\n\nFilesize : ${filesize}\n\nResolution: ${resolution}`
                megayaa.sendMessage(from, foto, image, {caption: cap})
                au = await getBuffer(result)
                megayaa.sendMessage(from, au, video, {mimetype: 'video/mp4', filename: `${title}.mp4`, quoted: lin, caption: `${title}`})
                break
            case 'wikipedia':
                q = body.slice(11)
                wiki = await axios.get(`https://lindow-python-api.herokuapp.com/api/wiki?q=${q}`)
                reply(`Hasil pencarin dari ${q}\n\n${wiki.data.result}\n\nJika undefined berarti query tidak ditemukan`)
                break
            case 'kusonime':
                try {
                q = body.slice(10)
                kus = await axios.get(`https://lindow-python-api.herokuapp.com/api/kuso?q=${q}`)
                var { info, link_dl, sinopsis, thumb, title } = kus.data
                buf = await getBuffer(thumb)
                cap = `Title : ${title}\n\n${info}\n\nLink download : ${link_dl}\n\nSinopsis : ${sinopsis}`
                megayaa.sendMessage(from, buf, image, {caption: cap})
                } catch (e) {
                console.log(e)
                reply(`Anime ${q} tidak ditemukan, coba cari title lain`)
                }
                break
            case 'dewabatch':
                try {
                q = body.slice(11)
                dew = await axios.get(`https://lindow-python-api.herokuapp.com/api/dewabatch?q=${q}`)
                var { result, sinopsis, thumb } = dew.data
                buffer = await getBuffer(thumb)
                cap = `${result}\n\n${sinopsis}`
                megayaa.sendMessage(from, buffer, image, {caption: cap})
                } catch (e) {
                console.log(e)
                reply(`Anime ${q} tidak dapat ditemukan`)
                }
                break
            case 'pinterest':
                getBuffer(`https://lindow-api.herokuapp.com/api/pinterest?search=${body.slice(11)}&apikey=${apikey}`).then((result) => {
                megayaa.sendMessage(from, result, image)
                })
                break
            case 'noprefix':
                prefix = ''
                reply('succes')
                break
            case 'tinyurl':
                url = args.join(" ")
                request(`https://tinyurl.com/api-create.php?url=${url}`, function (error, response, body) {
                try {
                    reply(body)
                  } catch (e) {
                    reply(e)
                  }
                })
                break
                case 'cuaca':
			cuaca = await fetchJson(`https://docs-jojo.herokuapp.com/api/cuaca?q=${args}`)	
			ingfo = `Tempat : ${cuaca.result.tempat}\nCuaca : ${cuaca.result.cuaca}\nDeskripsi : ${cuaca.result.desk}\nSuhu : ${cuaca.result.suhu}\nKelembapan : ${cuaca.result.kelembapan}\nUdara : ${cuaca.result.udara}\nAngin : ${cuaca.result.angin}`
			reply(ingfo)
	break
            case 'gempa':
                const tres = await Gempa()
                var { Waktu, Lintang, Bujur, Magnitude, Kedalaman, Wilayah, Map } = tres.result
                console.log(Map)
                captt = `Waktu : ${Waktu}\nLintang : ${Lintang}\nBujur : ${Bujur}\nWilayah : ${Wilayah}`
                thumbbb = await getBuffer(Map)
                megayaa.sendMessage(from, thumbbb, image, {caption: `${captt}`})
                break
case 'gempaterkini':                           	anu = await fetchJson(`https://bmkg-api.zhirrr.repl.co/api/gempa/terkini`)
teks = '❉────────────────❉\n'
					for (let i of anu.gempa_terkini) {
						teks += `*Waktu :* ${i.waktu_gempa}\n*Lintang :* ${i.lintang}\n*Bujur :* ${i.bujur}\n*Magnitudo :* ${i.magnitudo}\n*Kedalaman :* ${i.kedalaman}\n*Wilayah :* ${i.wilayah}\n❉────────────────❉\n`
					}
						reply(teks.trim())
					break
            case 'herolist':
                await herolist().then((ress) => {
                    let hm = `*Menampilkan list hero mobile legends*\n\n`
                    for (var i = 0; i < ress.hero.length; i++) {
                        hm += '➣  ' + ress.hero[i] + '\n'
                    }
                    reply(hm)
                    })
                break
            case 'herodetail':
                herodetails(body.slice(12)).then((res) => {
                capt = `*Hero details ${body.slice(12)}*
*Nama* : ${res.hero_name}
*Role* : ${res.role}
*Quotes* : ${res.entrance_quotes}
*Fitur Hero* : ${res.hero_feature}
*Spesial* : ${res.speciality}
*Rekomendasi Lane* : ${res.laning_recommendation}
*Harga* : ${res.price.battle_point} (Battle point) | ${res.price.diamond} (Diamond) | ${res.price.hero_fragment} (Hero Fragment)
*Tahun Rilis* : ${res.release_date}
*Skill* : 
*Durability* : ${res.skill.durability}
*Offence* : ${res.skill.offense}
*Skill Effect* : ${res.skill_effects}
*Difficulty* : ${res.skill.difficulty}
 
*Movement Speed* : ${res.attributes.movement_speed}
*Physical Attack* : ${res.attributes.physical_attack}
*Magic Defense* : ${res.attributes.magic_defense}
*Ability Crit Rate* : ${res.attributes.ability_crit_rate}
*HP* : ${res.attributes.hp}
*Mana* : ${res.attributes.mana}
*Mana Regen* : ${res.attributes.mana_regen}
*Story* : ${res.background_story}
`
                reply(capt)
                })
                break
            case 'kartundrama':
                ress = await Drama()
                let megg = `Random Drama Kartun`
                for (let i = 0; i < ress.hasil.length; i++) {
                  megg += `\n\n${ress.hasil[i].sinopsis}\nUrl : ${ress.hasil[i].url}`
                }
                thumb = await getBuffer(ress.hasil[0].img)
                megayaa.sendMessage(from, thumb, image, {caption: `${megg}`})
                break
            case 'kartunadventure':
                ress = await Adventure()
                let megggg = `Random Adventure Kartun`
                for (let i = 0; i < ress.hasil.length; i++) {
                  megggg += `\n\n${ress.hasil[i].sinopsis}\nUrl : ${ress.hasil[i].link}`
                }
                thumb = await getBuffer(ress.hasil[0].img)
                megayaa.sendMessage(from, thumb, image, {caption: `${megggg}`})
                break
            case 'kartunaction':
                ress = await Action()
                let meggg = `Random Action Kartun`
                for (let i = 0; i < ress.hasil.length; i++) {
                  meggg += `\n\n${ress.hasil[i].sinopsis}\nUrl : ${ress.hasil[i].link}`
                }
                thumb = await getBuffer(ress.hasil[0].img)
                megayaa.sendMessage(from, thumb, image, {caption: `${meggg}`})
                break
            case 'kartunmovie':
                try {
                result = await Movie()
                let meg = `Random Movie Kartun`
		for (let i = 0; i < result.hasil.length; i++) {
	        meg += `\n\n${result.hasil[i].sinopsis}\nUrl : ${result.hasil[i].url}`
	        }
		thumb = await getBuffer(result.hasil[0].img)
                megayaa.sendMessage(from, thumb, image, {caption: `${meg}`})
                } catch (e) {
                console.log(e)
                reply(e)
                }
                break
            case 'searchkartun':
                film = body.slice(14)
                try {
                    result = await SearchKartun(film)
		    let hehee = `Search kartun\nQuery : ${film}`
		    for (let i = 0; i < result.hasil.length; i++) {
		    hehee += `\n\n${result.hasil[i].sinopsis}\nLink : ${result.hasil[i].link}\nEpisode : ${result.hasil[i].episode}\nGenre : ${result.hasil[i].genre}`
		    }
		    thumb = await getBuffer(result.hasil[0].image)
                    megayaa.sendMessage(from, thumb, image, {caption: `${hehee}`})
                } catch (e) {
                console.log(e)
                reply(`Error, Coba judul lain!\n\nExample: ${prefix}searchkartun Spongebob`)
                }
		break
            case 'ayatkursi':
                res = await axios.get(`https://lindow-api.herokuapp.com/api/muslim/ayatkursi?apikey=${apikey}`)
                var { tafsir, arabic, latin } = res.data.result.data
                reply(`Tafsir : ${tafsir}\n\nArabic : ${arabic}\n\nLatin : ${latin}`)
                break
            case 'kisahnabi':
                nama = budy.slice(11)
                getres = await axios.get(`https://lindow-api.herokuapp.com/api/kisahnabi?nabi=${nama}&apikey=${apikey}`)
                var { nabi, lahir, umur, tempat, kisah } = getres.data.result.nabi
                caption = `Kisah Nabi\n\nNama nabi : ${nabi}\n\nLahir pada : ${lahir}\n\nUmur : ${umur}\n\nTempat : ${tempat}\n\nKisah :\n\n${kisah}`
                foto = await getBuffer(`${getres.data.result.nabi.image}`)
                megayaa.sendMessage(from, foto, image, {caption: caption})
                break
            case 'quoteislam':
                quote = await axios.get(`https://lindow-api.herokuapp.com/api/randomquote/muslim?apikey=${apikey}`)
                reply(`${quote.data.result.text_id}`)
                break
            case 'listimage':
	        teks = '*List Image :*\n\n'
                for (let awokwkwk of imagenye) {
		teks += `- ${awokwkwk}\n`
		}
		teks += `\n*Total : ${imagenye.length}*`
		megayaa.sendMessage(from, teks.trim(), extendedText, { quoted: lin, contextInfo: { "mentionedJid": imagenye } })
		break
            case 'getimage':
		namastc = body.slice(10)
		buffer = fs.readFileSync(`./lib/image/${namastc}.jpeg`)
		megayaa.sendMessage(from, buffer, image, {quoted: {
                    key: {
                        fromMe: false, participant: `0@s.whatsapp.net`, ...(from ? {
                        remoteJid: "status@broadcast"
                        }: {})
                    }, message: { conversation: `Result for database : ${namastc}.jpg` }}})
		break
            case 'addimage':
	        if (!isQuotedImage) return reply('reply image!')
	        svst = body.slice(10)
		if (!svst) return reply('input image name!')
	        boij = JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo
                delb = await megayaa.downloadMediaMessage(boij)
		imagenye.push(`${svst}`)
	        fs.writeFileSync(`./lib/image/${svst}.jpeg`, delb)
		fs.writeFileSync('./database/image.json', JSON.stringify(imagenye))
		    reply(`Success add image\n${prefix}listimage to view list image`)
		break
            case 'exif':
	        if (!itsMe) return reply('This command only for lindow')
	        if (args.length < 1) return reply(`Penggunaan ${prefix}exif nama|autho`)
		if (!arg.split('|')) return reply(`Penggunaan ${prefix}exif nama|author`)
		    exif.create(arg.split('|')[0], arg.split('|')[1])
		    reply('sukses')
	        break
            case 'takestick':
	        if (!isQuotedSticker) return reply(`Reply sticker dengan caption *${prefix}takestick nama|author*`)
		const pembawm = body.slice(11)
		if (!pembawm.includes('|')) return reply(`Reply sticker dengan caption *${prefix}takestick nama|author*`)
                const encmedia = JSON.parse(JSON.stringify(lin).replace('quotedM','m')).message.extendedTextMessage.contextInfo
                const media = await megayaa.downloadAndSaveMediaMessage(encmedia, `./sticker/${sender}`)
		const packname = pembawm.split('|')[0]
	        const author = pembawm.split('|')[1]
		    exif.create(packname, author, `takestick_${sender}`)
		    exec(`webpmux -set exif ./sticker/takestick_${sender}.exif ./sticker/${sender}.webp -o ./sticker/${sender}.webp`, async (error) => {
		    if (error) return reply('error')
		    wa.sendSticker(from, fs.readFileSync(`./sticker/${sender}.webp`), lin)
		    fs.unlinkSync(media)
		    fs.unlinkSync(`./sticker/takestick_${sender}.exif`)
		})
		break
            case 'scdl':
                var url = budy.slice(6)
                var res = await axios.get(`https://lindow-api.herokuapp.com/api/dlsoundcloud?url=${url}&apikey=${apikey}`)
                var { title, result } = res.data
                thumbb = await getBuffer(`${res.data.image}`)
                megayaa.sendMessage(from, thumbb, image, {caption: `${title}`})
                    audiony = await getBuffer(result)
                    megayaa.sendMessage(from, audiony, audio, {mimetype: 'audio/mp4', filename: `${title}.mp3`, quoted: lin})
                break
            case 'ppcouple':
                    getres = await axios.get(`https://lindow-api.herokuapp.com/api/ppcouple?apikey=${apikey}`)
                    var { male, female } = getres.data.result
                    picmale = await getBuffer(`${male}`)
                    megayaa.sendMessage(from, picmale, image)
                    picfemale = await getBuffer(`${female}`)
                    megayaa.sendMessage(from, picfemale, image)
                break
            case 'randomaesthetic':
                    url = `https://lindow-api.herokuapp.com/api/randomaesthetic?apikey=${apikey}`
                    estetik = await getBuffer(url)
                    megayaa.sendMessage(from, estetik, video, {mimetype: 'video/mp4', filename: `estetod.mp4`, quoted: lin, caption: 'success'})
                break
            case 'asupan':
                    url = `https://lindow-api.herokuapp.com/api/asupan?apikey=${apikey}`
                    asupan = await getBuffer(url)
                    megayaa.sendMessage(from, asupan, video, {mimetype: 'video/mp4', filename: `asupan.mp4`, quoted: lin, caption: 'success'})
                break
            case 'igdl':
                    var ini_url = body.slice(6)
                    var ini_url2 = await axios.get(`https://lindow-api.herokuapp.com/api/igdl?link=${ini_url}&apikey=${apikey}`)
                    var ini_url3 = ini_url2.data.result.url
                    var ini_type = image
                    if (ini_url3.includes(".mp4")) ini_type = video
                    var ini_buffer = await getBuffer(ini_url3)
                    var inicaption = `Username account : ${ini_url2.data.result.username}\n\nCaption : ${ini_url2.data.result.caption}\n\nShortcode : ${ini_url2.data.result.shortcode}\n\nDate : ${ini_url2.data.result.date}`
                    megayaa.sendMessage(from, ini_buffer, ini_type, {quoted: lin, caption: `${inicaption}`})
                break
	    case 'colong':
		if (!isQuotedSticker) return reply(`Reply sticker dengan caption *${prefix}colong*`)
		const encmediia = JSON.parse(JSON.stringify(lin).replace('quotedM','m')).message.extendedTextMessage.contextInfo
	        const meidia = await megayaa.downloadAndSaveMediaMessage(encmediia, `./sticker/${sender}`)
		    exec(`webpmux -set exif ./sticker/data.exif ./sticker/${sender}.webp -o ./sticker/${sender}.webp`, async (error) => {
		    if (error) return reply('error')
		    wa.sendSticker(from, fs.readFileSync(`./sticker/${sender}.webp`), lin)
		    fs.unlinkSync(meidia)
	        })
		break
            case 'swm':
	    case 'stickerwm':
	        if (isMedia && !lin.message.videoMessage || isQuotedImage) {
		if (!arg.includes('|')) return reply(`Kirim gambar atau reply gambar dengan caption *${prefix}stickerwm nama|author*`)
		const encmedia = isQuotedImage ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
		const media = await megayaa.downloadAndSaveMediaMessage(encmedia, `./sticker/${sender}`)
		const packname1 = arg.split('|')[0]
		const author1 = arg.split('|')[1]
		exif.create(packname1, author1, `stickwm_${sender}`)
		    await ffmpeg(`${media}`)
		    .input(media)
		    .on('start', function (cmd) {
		        console.log(`Started : ${cmd}`)
		    })
		    .on('error', function (err) {
		    console.log(`Error : ${err}`)
		fs.unlinkSync(media)
		reply('error')
		})
		.on('end', function () {
		console.log('Finish')
		exec(`webpmux -set exif ./sticker/stickwm_${sender}.exif ./sticker/${sender}.webp -o ./sticker/${sender}.webp`, async (error) => {
	        if (error) return reply('error')
	        wa.sendSticker(from, fs.readFileSync(`./sticker/${sender}.webp`), lin)
		    fs.unlinkSync(media)	
		    fs.unlinkSync(`./sticker/${sender}.webp`)	
		    fs.unlinkSync(`./sticker/stickwm_${sender}.exif`)
		    })
		})
		.addOutputOptions([`-vcodec`,`libwebp`,`-vf`,`scale='min(320,iw)':min'(320,ih)':force_original_aspect_ratio=decrease,fps=15, pad=320:320:-1:-1:color=white@0.0, split [a][b]; [a] palettegen=reserve_transparent=on:transparency_color=ffffff [p]; [b][p] paletteuse`])
		.toFormat('webp')
		.save(`./sticker/${sender}.webp`)
		} else if ((isMedia && lin.message.videoMessage.fileLength < 10000000 || isQuotedVideo && lin.message.extendedTextMessage.contextInfo.quotedMessage.videoMessage.fileLength < 10000000)) {
		if (!arg.includes('|')) return reply(`Kirim gambar atau reply gambar dengan caption *${prefix}stickerwm nama|author*`)
		const encmedia = isQuotedVideo ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
		const media = await megayaa.downloadAndSaveMediaMessage(encmedia, `./sticker/${sender}`)
		const packname1 = arg.split('|')[0]
		const author1 = arg.split('|')[1]
		    exif.create(packname1, author1, `stickwm_${sender}`)
		    reply('wait')
		    await ffmpeg(`${media}`)
		        .inputFormat(media.split('.')[4])
			.on('start', function (cmd) {
			console.log(`Started : ${cmd}`)
		    })
		    .on('error', function (err) {
		    console.log(`Error : ${err}`)
		        fs.unlinkSync(media)
			tipe = media.endsWith('.mp4') ? 'video' : 'gif'
			reply('error')
		    })
		    .on('end', function () {
		    console.log('Finish')
		        exec(`webpmux -set exif ./sticker/stickwm_${sender}.exif ./sticker/${sender}.webp -o ./sticker/${sender}.webp`, async (error) => {
			if (error) return reply('error')
			wa.sendSticker(from, fs.readFileSync(`./sticker/${sender}.webp`), lin)									
			fs.unlinkSync(media)
			fs.unlinkSync(`./sticker/${sender}.webp`)
			fs.unlinkSync(`./sticker/stickwm_${sender}.exif`)
			})
		    })
		    .addOutputOptions([`-vcodec`,`libwebp`,`-vf`,`scale='min(320,iw)':min'(320,ih)':force_original_aspect_ratio=decrease,fps=15, pad=320:320:-1:-1:color=white@0.0, split [a][b]; [a] palettegen=reserve_transparent=on:transparency_color=ffffff [p]; [b][p] paletteuse`])
		    .toFormat('webp')
		    .save(`./sticker/${sender}.webp`)
		} else {
		reply(`Kirim gambar/video dengan caption ${prefix}stickerwm nama|author atau tag gambar/video yang sudah dikirim\nNote : Durasi video maximal 10 detik`)
	        }
		break
            case 'sticker':
	    case 'stiker':
	    case 's':
		if (isMedia && !lin.message.videoMessage || isQuotedImage) {
		const encmedia = isQuotedImage ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
		const media = await megayaa.downloadAndSaveMediaMessage(encmedia, `./sticker/${sender}`)
		await ffmpeg(`${media}`)
		.input(media)
		.on('start', function (cmd) {
	        console.log(`Started : ${cmd}`)
		})
		.on('error', function (err) {
		console.log(`Error : ${err}`)
  		fs.unlinkSync(media)
		reply('error')
		})
		.on('end', function () {
		console.log('Finish')
		exec(`webpmux -set exif ./sticker/data.exif ./sticker/${sender}.webp -o ./sticker/${sender}.webp`, async (error) => { 
                if (error) return reply('error')
		    wa.sendSticker(from, fs.readFileSync(`./sticker/${sender}.webp`), lin)
		    fs.unlinkSync(media)	
		    fs.unlinkSync(`./sticker/${sender}.webp`)	
		    })
		})
		.addOutputOptions([`-vcodec`,`libwebp`,`-vf`,`scale='min(320,iw)':min'(320,ih)':force_original_aspect_ratio=decrease,fps=15, pad=320:320:-1:-1:color=white@0.0, split [a][b]; [a] palettegen=reserve_transparent=on:transparency_color=ffffff [p]; [b][p] paletteuse`])
		.toFormat('webp')
		.save(`./sticker/${sender}.webp`)
		} else if ((isMedia && lin.message.videoMessage.fileLength < 10000000 || isQuotedVideo && lin.message.extendedTextMessage.contextInfo.quotedMessage.videoMessage.fileLength < 10000000)) {
		    const encmedia = isQuotedVideo ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
		    const media = await megayaa.downloadAndSaveMediaMessage(encmedia, `./sticker/${sender}`)
		    reply('wait')
			await ffmpeg(`${media}`)
			.inputFormat(media.split('.')[4])
			.on('start', function (cmd) {
			console.log(`Started : ${cmd}`)
		})
		.on('error', function (err) {
		console.log(`Error : ${err}`)
		    fs.unlinkSync(media)
		    tipe = media.endsWith('.mp4') ? 'video' : 'gif'
		    reply('error')
		})
		.on('end', function () {
		console.log('Finish')
		exec(`webpmux -set exif ./sticker/data.exif ./sticker/${sender}.webp -o ./sticker/${sender}.webp`, async (error) => {
		if (error) return reply('error')
	            wa.sendSticker(from, fs.readFileSync(`./sticker/${sender}.webp`), lin)
		    fs.unlinkSync(media)
		    fs.unlinkSync(`./sticker/${sender}.webp`)
		    })
		})
		.addOutputOptions([`-vcodec`,`libwebp`,`-vf`,`scale='min(320,iw)':min'(320,ih)':force_original_aspect_ratio=decrease,fps=15, pad=320:320:-1:-1:color=white@0.0, split [a][b]; [a] palettegen=reserve_transparent=on:transparency_color=ffffff [p]; [b][p] paletteuse`])
		.toFormat('webp')
		.save(`./sticker/${sender}.webp`)
	        } else {
		reply(`Kirim gambar/video dengan caption ${prefix}sticker atau tag gambar/video yang sudah dikirim\nNote : Durasi video maximal 10 detik`)
		}
	        break
            case 'getbio':
                var yy = lin.message.extendedTextMessage.contextInfo.mentionedJid[0]
                var p = await megayaa.getStatus(`${yy}`, MessageType.text)
                reply(p.status)
                if (p.status == 401) {
                reply("Status Profile Not Found")
                }
                break
	   case 'getpic':
		if (lin.message.extendedTextMessage != undefined){
		mentioned = lin.message.extendedTextMessage.contextInfo.mentionedJid
	        try {
		    pic = await megayaa.getProfilePicture(mentioned[0])
		} catch {
		    pic = 'https://i.ibb.co/Tq7d7TZ/age-hananta-495-photo.png'
		}
		thumb = await getBuffer(pic)
		megayaa.sendMessage(from, thumb, image, {caption: 'success'})
	        }
		break
            case 'fdeface': 
		var nn = budy.slice(9)
                var urlnye = nn.split("|")[0];
                var titlenye = nn.split("|")[1];
	        var descnye = nn.split("|")[2];
                run = getRandom('.jpeg')
                var media1 = isQuotedImage ? JSON.parse(JSON.stringify(lin).replace('quotedM','m')).message.extendedTextMessage.contextInfo : lin
                var media2 = await megayaa.downloadAndSaveMediaMessage(media1)
                var ddatae = await imageToBase64(JSON.stringify(media2).replace(/\"/gi, ''))
                megayaa.sendMessage(from, {
                    text: `${urlnye}`,
                    matchedText: `${urlnye}`,
                    canonicalUrl: `${urlnye}`,
                    description: `${descnye}`,
                    title: `${titlenye}`,
                    jpegThumbnail: ddatae }, 'extendedTextMessage', { detectLinks: false })
		break
            case 'setbio':
	        if (!itsMe) return reply('This command only for lindow')
		if (!arg) return reply('masukkan bio')
	        wa.setBio(arg)
	        .then((res) => wa.sendFakeStatus2(from, JSON.stringify(res), fake))
		.catch((err) => wa.sendFakeStatus2(from, JSON.stringify(err), fake))
		break
            case 'setname':
		if (!itsMe) return reply('This command only for lindow')
	        if (!arg) return reply('masukkan nama')
		wa.setName(arg)
		.then((res) => wa.sendFakeStatus2(from, JSON.stringify(res), fake))
		.catch((err) => wa.sendFakeStatus2(from, JSON.stringify(err), fake))
	        break
            case 'setreply':
		if (!itsMe) return reply('This command only for lindow')
	        if (!arg) return reply(`Penggunaan ${prefix}setreply teks`)
		fake = arg
		wa.sendFakeStatus2(from, `Sukses`, fake)
		break
            case 'term':
	        if (!itsMe) return reply('This command only for lindow')
		if (!arg) return
		exec(arg, (err, stdout) => {
		    if (err) return wa.sendFakeStatus2(from, err, fake)
		    if (stdout) wa.sendFakeStatus2(from, stdout, fake)
		})
		break
            case 'sendkontak':
	        if (!itsMe) return reply('This command only for lindow')
	        argz = arg.split('|')
	        if (!argz) return reply(`Penggunaan ${prefix}kontak @tag atau nomor|nama`)
		if (lin.message.extendedTextMessage != undefined){
                mentioned = lin.message.extendedTextMessage.contextInfo.mentionedJid
		wa.sendKontak(from, mentioned[0].split('@')[0], argz[1])
	        } else {
		wa.sendKontak(from, argz[0], argz[1])
                }
		break
            case 'speed': 
            case 'ping':
		let timestamp = speed();
		let latensi = speed() - timestamp
		wa.sendFakeStatus2(from, `Speed: ${latensi.toFixed(4)}second`, fake)
		break
            case 'runtime':
		run = process.uptime()
		let text = msg.runtime(run)
	        wa.sendFakeStatus2(from, MessageType.text,`Runtime bro`)
		break
            case 'unpin':
                if (!itsMe) return reply('This command only for lindow')
                megayaa.modifyChat(from, ChatModification.unpin)
                reply('*succes unpin this chat*')
                console.log('unpin chat = ' + from)
                break
            case 'pin':
                if (!itsMe) return reply('This command only for lindow')
                megayaa.modifyChat(from, ChatModification.pin)
                reply('*succes pin this chat*')
                console.log('pinned chat = ' + from)
                break
            case 'unread?':
		const unread = await megayaa.loadAllUnreadMessages()
	        megayaa.sendMessage(from, `unread message count : *${unread.length}*`, text)
                break
            case 'unarchiveall':
                if (!itsMe) return reply('This command only for mega')
                reply('*succes unarchive all chat*')
                console.log('succes unarchive chat = ' + from)
                anu = await megayaa.chats.all()
                for (let _ of anu) {
                megayaa.modifyChat(_.jid, ChatModification.unarchive)
                }
                break
            case 'archive':
                if (!itsMe) return reply('This command only for lindow')
                reply('*okey wait..*')
                console.log('succes archive chat = ' + from)
                await sleep(3000)
                megayaa.modifyChat(from, ChatModification.archive)
                break
            case 'delthischat':
                if (!itsMe) return reply('This command only for lindow')
                reply('*succes delete this chat*')
                console.log('succes delete chat = ' + from)
                await sleep(4000)
                megayaa.modifyChat(from, ChatModification.delete)
                break
            case 'mute':
                if (!itsMe) return reply('This command only for mega')
                megayaa.modifyChat(from, ChatModification.mute, 24*60*60*1000)
                reply('*succes mute this chat*')
                console.log('succes mute chat = ' + from)
                break
            case 'unmute':
                if (!itsMe) return reply('This command only for mega')
                megayaa.modifyChat(from, ChatModification.unmute)
                reply('*succes unmute this chat*')
                console.log('succes unmute chat = ' + from)
                break
            case 'ytsearch':
                ytsr = require('ytsr')
                if (!args.length) return reply('input title!')
                try {
                    const input = args.join(" ")
                    const filter1 = await ytsr.getFilters(input)
                    const filters1 = filter1.get('Type').get('Video')
                    const { items } = await ytsr(filters1.url, { limit: 10 })
                    let hehe = `*YOUTUBE SEARCH*\n\n*Search Query:* ${input}\n`
                    for (let i = 0; i < items.length; i++) {
                        hehe += `\n\n=====================\n\n*Judul:* ${items[i].title}\n\n*ID:* ${items[i].id}\n\n*Viewers:* ${items[i].views}\n\n*Duration:* ${items[i].duration}\n\n*Link:* ${items[i].url}\n`
                    }
                    thumb = await getBuffer(items[0].bestThumbnail.url)
                    await megayaa.sendMessage(from, thumb, image, {quoted: lin, caption: `${hehe}\n\nDownload:\n${prefix}ytmp3 [link youtube] = Audio\n${prefix}ytmp4 [link youtube] = Video`})
                } catch(e) {
                    reply('Didn\'t find anything or there is any error!')
                    reply(`Error: ${e.message}`)
                }
                break
            case 'upstorypic':
                if (!itsMe) return reply('This command only for mega')
                var teksyy = body.slice(12)
                    reply('wait')
                var foto = isQuotedImage ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
		var inisiap = await megayaa.downloadAndSaveMediaMessage(foto)
                var inisiap2 = fs.readFileSync(inisiap)
                megayaa.sendMessage('status@broadcast', inisiap2, image, {quoted: lin, caption: `${teksyy}`})
                    reply('Succes!')
                break
            case 'upstoryvid':
                if (!itsMe) return reply('This command only for mega')
                reply('wait')
                var foto = isQuotedVideo ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
		var inisiap = await megayaa.downloadAndSaveMediaMessage(foto)
                var inisiap2 = fs.readFileSync(inisiap)
                megayaa.sendMessage('status@broadcast', inisiap2, video, {quoted: lin, caption: `${body.slice(12)}`})
                    reply('Succes!')
                break
            case 'upstory':
                if (!itsMe) return reply('This command only for mega')
                var teks = body.slice(9)
                megayaa.sendMessage('status@broadcast', teks, text)
                    reply('succses')
                break
            case 'unreadall':
                if (!itsMe) return reply('This command only for mega')
                var chats = await megayaa.chats.all()
                chats.map( async ({ jid }) => {
                await megayaa.chatRead(jid, 'unread')
                    })
		    var teks = `\`\`\`Successfully unread ${chats.length} chats !\`\`\``
		    await megayaa.sendMessage(from, teks, MessageType.text, {quoted: lin})
		    console.log(chats.length)
	        break
            case 'readall':
                if (!itsMe) return reply('This command only for mega')
                var chats = await megayaa.chats.all()
                chats.map( async ({ jid }) => {
                await megayaa.chatRead(jid)
                })
		var teks = `\`\`\`Successfully read ${chats.length} chats !\`\`\``
	        await megayaa.sendMessage(from, teks, MessageType.text, {quoted: lin})
		console.log(chats.length)
		break
            case 'fakereply':
		if (!args) return reply(`Usage :\n${prefix}fakereply [62xxx|pesan|balasanbot]]\n\nEx : \n${prefix}fakereply 0|hai|hai juga`)
		var ghh = budy.slice(11)
		var nomorr = ghh.split("|")[0];
	        var target = ghh.split("|")[1];
		var bot = ghh.split("|")[2];
	            megayaa.sendMessage(from, `${bot}`, MessageType.text, {quoted: { key: { fromMe: false, participant: nomorr+'@s.whatsapp.net', ...(from ? { remoteJid: from } : {}) }, message: { conversation: `${target}` }}})
                break
            case 'fordward':
	        megayaa.sendMessage(from, `${budy.slice(10)}`, MessageType.text, {contextInfo: { forwardingScore: 508, isForwarded: true }})
                break
            case 'tagall':
                if (!isAdmin) return reply('only for admin group')
                members_id = []
		teks = (args.length > 1) ? budy.slice(8).trim() : ''
	        teks += '\n\n'
	        for (let mem of groupMembers) {
		teks += `┣➥ @${mem.jid.split('@')[0]}\n`
		members_id.push(mem.jid)
		}
		mentions(teks, members_id, true)
		break
            case 'chat':
                if (!itsMe) return reply('This command only for lindow')
                var pc = budy.slice(6)
                var nomor = pc.split("|")[0];
                var org = pc.split("|")[1];
                megayaa.sendMessage(nomor+'@s.whatsapp.net', org, MessageType.text)   
                reply('done..')
                break
case 'chat1':
nom = body.slice(6)
nomer = nom.split("|")[0]
pesan = nom.split("|")[1]
megayaa.sendMessage(nomer + "@s.whatsapp.net", pesan, text)
reply(`Berhasil mengirimkan pesan ke nomor ${nomer}`)
break
            case 'setpp':
                if (!itsMe) return reply('This command only for lindow')
                megayaa.updatePresence(from, Presence.composing) 
                if (!isQuotedImage) return reply(`Kirim gambar dengan caption ${prefix}setpp atau tag gambar yang sudah dikirim`)
	        var media1 = JSON.parse(JSON.stringify(lin).replace('quotedM','m')).message.extendedTextMessage.contextInfo
		var media2 = await megayaa.downloadAndSaveMediaMessage(media1)
	        await megayaa.updateProfilePicture(meNumber, media2)
		reply('Done!')
	        break
            case 'kick':
                if (!isAdmin) return reply('this command only for admin')
	        if (!args) return reply(`Penggunaan ${prefix}kick @tag atau nomor`)
                if (lin.message.extendedTextMessage != undefined){
                mentioned = lin.message.extendedTextMessage.contextInfo.mentionedJid
		await wa.FakeTokoForwarded(from, `Bye...`, fake)
		    wa.kick(from, mentioned)
		} else {
	        await wa.FakeTokoForwarded(from, `Bye...`, fake)
		wa.kick(from, [args[0] + '@s.whatsapp.net'])
		}
		break
            case 'add':
                if (!isAdmin) return reply('only for admin group')
		if (!args) return reply(`Penggunaan ${prefix}add 628xxxx`)
		wa.add(from, [args[0] + '@s.whatsapp.net'])
                wa.FakeTokoForwarded(from, `Sukses`, fake)
                break
            case 'spam':
                if (!itsMe) return reply('This command only for mega')
	        if (!arg) return reply(`Penggunaan ${prefix}spam teks|jumlahspam`)
	        argz = arg.split("|")
		if (!argz) return reply(`Penggunaan ${prefix}spam teks|jumlah`)
                if (isNaN(argz[1])) return reply(`harus berupa angka`)
	        for (let i = 0; i < argz[1]; i++){
                megayaa.sendMessage(from, argz[0], MessageType.text)
		}
	        break
            case 'shutdown':
                if (!itsMe) return reply('This command only for megaa')
	        await wa.FakeTokoForwarded(from, `Bye...`, fake)
		await sleep(5000)
                megayaa.close()
		break
            case 'ocr': 
	        if ((isMedia && !lin.message.videoMessage || isQuotedImage) && args.length == 0) {
	    	var media1 = isQuotedImage ? JSON.parse(JSON.stringify(lin).replace('quotedM','m')).message.extendedTextMessage.contextInfo : lin
                var media2 = await megayaa.downloadAndSaveMediaMessage(media1)
                reply("*waitt*")
	    	await recognize(media2, {lang: 'eng+ind', oem: 1, psm: 3})
		    .then(teks => {
		    reply(teks.trim())
		    fs.unlinkSync(media2)
		})
		.catch(err => {
		reply(err.message)
		fs.unlinkSync(media2)
		})
	        } else {
		reply(`Send image and reply with caption ${prefix}ocr`)
		}
	        break
            case 'demoteall':
                members_id = []
		for (let mem of groupMembers) {
	   	members_id.push(mem.jid)
	  	}
                megayaa.groupDemoteAdmin(from, members_id)
                break
            case 'bugimg':
                var nnn = budy.slice(12)
                var urlnyee = nnn.split("|")[0];
                var titlenyee = nnn.split("|")[1];
                var descnyee = nnn.split("|")[2];
                var run = help.getRandomExt('.jpeg')
                var media1 = isQuotedImage ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
                var media2 = await megayaa.downloadAndSaveMediaMessage(media1)
                var ddatae = await imageToBase64(JSON.stringify(media2).replace(/\"/gi, ''))
                megayaa.sendMessage(from, {
                    text: `${body.slice(8)}`,
                    matchedText: `${urlnyee}`,
                    canonicalUrl: `${urlnyee}`,
                    description: `${descnyee}`,
                    title: `${titlenyee}`,
                    jpegThumbnail: ddatae
                    }, 'extendedTextMessage', { detectLinks: false})
                megayaa.sendMessage(from, 'Coba reply tuh', MessageType.text)
                break
            case 'public':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                if (public) return await reply('already in public mode')
                config["public"] = true
                public = true
                fs.writeFileSync("./config.json", JSON.stringify(config, null, 4))
                await wa.sendFakeStatus(from, "*Success changed to public mode*", "Public : true")
                break
            case 'self':
                if (!isOwner && !itsMe) return await reply('This command only for mega or owner')
                if (!public) return await reply('mode private is already')
                config["public"] = false
                public = false
                fs.writeFileSync("./config.json", JSON.stringify(config, null, 4))
                await wa.sendFakeStatus(from, "*Success changed to self mode*", "Self : true")
                break
            case 'setprefix':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                var newPrefix = args[0] || ""
                prefix = newPrefix
                await reply("Success change prefix to: " + prefix)
                break
            case 'broadcast':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                text = args.join(" ")
                for (let chat of totalChat) {
                    await wa.sendMessage(chat.jid, text)
                }
                break
case 'setthumb':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                /*if (!isQuotedImage && !isImage) return await reply('Gambarnya mana?')*/
                media1 = JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo
                mediaa = await megayaa.downloadMediaMessage(media1)
                fs.writeFileSync(`./lib/image/foto.jpg`, mediaa)
                await wa.sendFakeStatus(from, "*Succes changed image for help image*", "success")
                break
            case 'fakethumb':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                if (!isQuotedImage && !isImage) return await reply('reply image!')
                media = isQuotedImage ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
                media = await megayaa.downloadMediaMessage(media)
                await wa.sendFakeThumb(from, media)
                break
            case 'stats':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                texxt = await msg.stats(totalChat)
                await wa.sendFakeStatus(from, texxt, "BOT STATS")
                break
            case 'block':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                if (isGroup) {
                    if (mentionUser.length == 0) return await reply("tag target!")
                    return await wa.blockUser(sender, true)
                }
                await wa.blockUser(sender, true)
                break
            case 'unblock':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                if (isGroup) {
                    if (mentionUser.length == 0) return await reply("Tag targer!")
                    return await wa.blockUser(sender, false)
                }
                await wa.blockUser(sender, false)
                break
            case 'leave':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                if (!isGroup) return await reply('This command only for group baka')
                reply(`Akan keluar dari group ${groupName} dalam 3 detik`).then(async() => {
                    await help.sleep(3000)
                    await megayaa.groupLeave(from)
                })
                break
            case 'join':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                if (isGroup) return await reply('This command only for private chat')
                if (args.length == 0) return await reply('Link group?')
                var link = args[0].replace("https://chat.whatsapp.com/", "")
                await megayaa.acceptInvite(link)
                break
            case 'clearall':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                for (let chat of totalChat) {
                    await megayaa.modifyChat(chat.jid, "delete")
                }
                await wa.sendFakeStatus(from, "Success clear all chat", "success")
                break

            /** Group **/
            case 'hidetag':
                if (!isOwner && !itsMe) return await reply('This command only for owner or mega')
                if (!isAdmin && !isOwner && !itsMe) return await reply('this command only for admin, baka!')
                await wa.hideTag(from, args.join(" "))
                break
            case 'imagetag':
                if (!isGroup) return await reply('this command only for group')
                if (!isAdmin && !isOwner && !itsMe) return await reply('this command only for admin, baka!')
                if (!isQuotedImage && !isImage) return await reply(`Send image, and reply with caption ${prefix}imagetag`)
                media = isQuotedImage ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
                buffer = await megayaa.downloadMediaMessage(media)
                await wa.hideTagImage(from, buffer)
                break
            case 'toimg':
	        if (!isQuotedSticker) return reply(`send sticker and reply with caption ${prefix}toimg`)
	        if (lin.message.extendedTextMessage.contextInfo.quotedMessage.stickerMessage.isAnimated === true){
		reply(`Maaf tidak mendukung sticker gif`)
	        } else {
		var media1 = JSON.parse(JSON.stringify(lin).replace('quotedM','m')).message.extendedTextMessage.contextInfo
	        var media2 = await megayaa.downloadAndSaveMediaMessage(media1)
		ran = getRandom('.png')
                exec(`ffmpeg -i ${media2} ${ran}`, (err) => {
		fs.unlinkSync(media2)
		if (err) {
			reply(`error\n\n${err}`)
			fs.unlinkSync(ran)
			} else {
			buffer = fs.readFileSync(ran)
			megayaa.sendMessage(from, buffer, MessageType.image, {quoted: lin, caption: 'success'})
			fs.unlinkSync(ran)
			}
	            })
		}
		break
            case 'toptt':
		reply(`wait..`)
		var media1 = JSON.parse(JSON.stringify(lin).replace('quotedM','m')).message.extendedTextMessage.contextInfo
		var media2 = await megayaa.downloadAndSaveMediaMessage(media1)
		var ran = getRandom('.mp3')
		exec(`ffmpeg -i ${media2} ${ran}`, (err) => {
	        fs.unlinkSync(media2)
		if (err) return reply('error')
	        topt = fs.readFileSync(ran)
		megayaa.sendMessage(from, topt, audio, {mimetype: 'audio/mp4', quoted: lin, ptt:true})
	        })
		break
            case 'stickertag':
                if (!isGroup) return await reply('this command only for group')
                if (!isAdmin && !isOwner && !itsMe) return await reply('This command only for admin')
                if (!isQuotedImage && !isImage) return await reply('Stickernya mana?')
                media = isQuotedSticker ? JSON.parse(JSON.stringify(lin).replace('quotedM', 'm')).message.extendedTextMessage.contextInfo : lin
                buffer = await megayaa.downloadMediaMessage(media)
                await wa.hideTagSticker(from, buffer)
                break
            case 'promote':
                if (!isGroup) return await reply('this command only for group')
                if (!isAdmin) return await reply('This command only for admin')
                if (!botAdmin) return await reply('jadikan bot admin')
                if (mentionUser.length == 0) return await reply('Tag member')
                await wa.promoteAdmin(from, mentionUser)
                await reply(`Success promote member`)
                break
            case 'demote':
                if (!isGroup) return await reply('this command only for group')
                if (!isAdmin) return await reply('This command only for admin')
                if (!botAdmin) return await reply('This command is available if the bot admin')
                if (mentionUser.length == 0) return await reply('Tag member!')
                await wa.demoteAdmin(from, mentionUser)
                await reply(`Success demote member`)
                break
            case 'admin':
                var textt = msg.admin(groupAdmins, groupName)
                await wa.sendFakeStatus(from, textt, "LIST ADMIN", groupAdmins)
                break
            case 'linkgc':
                var link = await wa.getGroupInvitationCode(from)
                await wa.sendFakeStatus(from, link, "This link group")
                break
            case 'group':
                if (!isGroup) return await reply('this command only for group')
                if (!isAdmin) return await reply('This command only for admin')
                if (!botAdmin) return await reply('This command is available if the bot admin')
                if (args[0] === 'open') {
                    megayaa.groupSettingChange(from, GroupSettingChange.messageSend, false).then(() => {
                        wa.sendFakeStatus(from, "*Success open group*", "GROUP SETTING")
                    })
                } else if (args[0] === 'close') {
                    megayaa.groupSettingChange(from, GroupSettingChange.messageSend, true).then(() => {
                        wa.sendFakeStatus(from, "*Succes close group*", "GROUP SETTING")
                    })
                } else {
                    await reply(`Example: ${prefix}${command} open/close`)
                }
                break
            case 'setnamegc':
                if (!isGroup) return await reply('this command only for groups')
                if (!isAdmin) return await reply('This command only for admin')
                if (!botAdmin) return await reply('This command is available if the bot admin')
                var newName = args.join(" ")
                megayaa.groupUpdateSubject(from, newName).then(() => {
                    wa.sendFakeStatus(from, "Succes change subject name to" + newName, "GROUP SETTING")
                })
                break
            case 'setdesc':
                if (!isGroup) return await reply('This command only for groups')
                if (!isAdmin) return await reply('This command only for admin')
                if (!botAdmin) return await reply('This command is available if the bot admin')
                var newDesc = args.join(" ")
                megayaa.groupUpdateDescription(from, newDesc).then(() => {
                    wa.sendFakeStatus(from, "Succes change description group to" + newDesc, "GROUP SETTING")
                })
            default:
if (chats == 'bot') {
                  tujuh = fs.readFileSync('./mp3/PTT-20210315-WA0057.m4a');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'sayang') {
                  tujuh = fs.readFileSync('./mp3/apasayang.ogg');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'ara ara') {
                  tujuh = fs.readFileSync('./mp3/ara_ara.mp3');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'haha') {
                  tujuh = fs.readFileSync('./mp3/hahaha.mp3');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'fbi') {
                  tujuh = fs.readFileSync('./mp3/fbi.mp3');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'lama') {
                  tujuh = fs.readFileSync('./mp3/lama.mp3');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'lag') {
                  tujuh = fs.readFileSync('./mp3/windowsxp.mp3');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'hmm') {
                  tujuh = fs.readFileSync('./mp3/hergo.mp3');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'always love you') {
                  tujuh = fs.readFileSync('./mp3/alwaysloveyou.mp3');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
if (chats == 'Assalamualaikum') {
                  tujuh = fs.readFileSync('./mp3/waalaikumsalam.ogg');
                  megayaa.sendMessage(from, tujuh, MessageType.audio, {quoted: lin, mimetype: 'audio/mp4', ptt:true})
                  }
                if (body.startsWith(">")) {
                    if (!itsMe) return await reply('This command only for meguy')
                    return await reply(JSON.stringify(eval(args.join(" ")), null, 2))
                }
        }
    } catch (e) {
        console.log(chalk.whiteBright("├"), chalk.keyword("aqua")("[  ERROR  ]"), chalk.keyword("red")(e))
    }
})
© 2021 GitHub, Inc.
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
Loading complete
