# Transcribe.fm

Transcribe.fm is a lightning-fast transcription service for podcasters. We can transcribe an hour of audio to in just 30 seconds with speaker diarization & word-by-word fidelity. Pay what you want in Satoshis using a Lightning wallet.

## FAQ

### How accurate is it?
Our ASR proivder measured their own accuracy at 90.5% compared to OpenAI’s Whisper model at 83.8-87.6%

### How fast is it?
It can perform at 120x real-time speed, which means that you can transcribe an hour of audio in just 30 seconds.

### How much does it cost?
We offer a flexible pay-what-you-want model. To cover our costs, we’ve set a small minimum based on the duration of the audio.

### Do you offer an API?
Yes! We support the new [L402 Protocol](https://docs.lightning.engineering/the-lightning-network/l402), previously known as LSAT, to charge Lightning payments for API requests.

## API

> [!NOTE]  
> To see a demo of this process, including paying the invoice with webln, check out the [API example](https://github.com/transcribefm/api-example) repo or [run on Replit](https://replit.com/new/github/transcribefm/api-example) yourself.

### Requesting an invoice
To request an invoice, submit an `audio_url` to `https://transcribe.fm/api/v1/transcribe`.

```js
fetch("https://transcribe.fm/api/v1/transcribe", {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    "audio_url": "https://podnews.net/audio/podnews230717.mp3"
  })
})
```

In response, the server will reply with a 402 (Payment Required) status code and a `WWW-Authenticate` header ([[RFC 7235], Section 4.1](https://tools.ietf.org/html/rfc7235#section-4.1)), containing a `macaroon` and a Lightning invoice, like this:
```
HTTP/1.1 402 Payment Required

Date: Mon, 04 Feb 2014 16:50:53 GMT

WWW-Authenticate: L402 macaroon="AGIAJEemVQUTEyNCR0exk7ek90Cg==", invoice="lnbc1500n1pw5kjhmpp5fu6xhthlt2vucmzkx6c7wtlh2r625r30cyjsfqhu8rsx4xpz5lwqdpa2fjkzep6yptksct5yp5hxgrrv96hx6twvusycn3qv9jx7ur5d9hkugr5dusx6cqzpgxqr23s79ruapxc4j5uskt4htly2salw4drq979d7rcela9wz02elhypmdzmzlnxuknpgfyfm86pntt8vvkvffma5qc9n50h4mvqhngadqy3ngqjcym5a"
```
The client can then pay the Lightning invoice to reveal the `preimage`. 

### Requesting transcription
By including the `macaroon` and the `preimage` in an Authorization header, you can call the `/transcribe` endpoint again. This will return a `transcript_id`, while the audio is being transcribed. To retreive the transcript itself, a subsequent request is needed.

```js
fetch("https://transcribe.fm/api/v1/transcribe", {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: 'L402 AGIAJEemVQUTEyNCR0exk7ek90Cg==:1234abcd1234abcd1234abcd'
  },
  body: JSON.stringify({
    "audio_url": "https://podnews.net/audio/podnews230717.mp3"
  })
})
```

### Downloading a transcript
With a `transcript_id`, you can request a transcript by including an Accept header with a supported MIME type. The value must be one one the following: 
* `application/json`
* `application/x-subrip`
* `text/vtt`
* `text/plain`

```js
fetch(`https://transcribe.fm/api/v1/download/${transcript_id}`, {
  method: 'GET',
  headers: {
    Accept: 'text/plain'
  }
});
```
