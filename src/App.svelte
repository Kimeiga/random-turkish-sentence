<!-- a284151a462b5b9bd2448ff75d2fe7744f14d394 -->
<!-- api key for nlp cloud -->

<script>
  let voice = window.speechSynthesis
    .getVoices()
    .filter((v) => v.lang.startsWith("zh"))[0];

  var decodeEntities = (function () {
    // this prevents any overhead from creating the object each time
    var element = document.createElement("div");

    function decodeHTMLEntities(str) {
      if (str && typeof str === "string") {
        // strip script/html tags
        str = str.replace(/<script[^>]*>([\S\s]*?)<\/script>/gim, "");
        str = str.replace(/<\/?\w(?:[^"'>]|"[^"]*"|'[^']*')*>/gim, "");
        element.innerHTML = str;
        str = element.textContent;
        element.textContent = "";
      }

      return str;
    }

    return decodeHTMLEntities;
  })();

  // import pos from "pos";
  // import chunker from "pos-chunker";


  function makeid(length) {
    let result = "";
    let characters =
      "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
    let charactersLength = characters.length;
    for (let i = 0; i < length; i++) {
      result += characters.charAt(Math.floor(Math.random() * charactersLength));
    }
    return result;
  }
  let translationText;
  let turkishText;
  // let chineseText;
  // let traditionalChineseText;
  let pinyinTextList;
  $: rubyTexts = [];

  function SetBackgroundImage() {
    // let words = new pos.Lexer().lex(translationText);
    // let tags = new pos.Tagger()
    //   .tag(words)
    //   .map(function (tag) {
    //     return tag[0] + "/" + tag[1];
    //   })
    //   .join(" ");

    // // debugger;
    // let nouns = chunker.chunk(tags, "[{ tag: NN|NNP|VBG }]");
    // // console.log(nouns);
    // let extractedNouns = [];
    // let nouns2 = [...nouns.matchAll(/\{([^}]+)\}/g)];
    // // console.log(nouns2);

    // for (let n of nouns2) {
    //   // console.log(n);
    //   extractedNouns.push(n[1].match(/(.*?)\//)[1]);
    // }

    // console.log("extractedNouns");
    // console.log(extractedNouns);

    // remove short words
    let extractedWords = translationText.split(" ").filter((x) => x.length > 4);

    if (extractedWords.length == 0) {
      extractedWords = translationText.split(" ").filter((x) => x.length > 3);
    }

    if (extractedWords.length == 0) {
      extractedWords = translationText.split(" ").filter((x) => x.length > 2);
    }

    // remove words with apostrophe's like "didn't"
    let noApostrophes = extractedWords.filter((x) => x.indexOf("'") == -1);
    if (noApostrophes) extractedWords = noApostrophes;

    console.log("extractedWords");
    console.log(extractedWords);

    let backgroundImageURL = `url("https://source.unsplash.com/random/?${extractedWords.join()}")`;

    // console.log(backgroundImageURL);

    document.body.style.backgroundImage = backgroundImageURL;
  }

  // function SetBackgroundImage() {
  //   let words = new pos.Lexer().lex(translationText);
  //   let tags = new pos.Tagger()
  //     .tag(words)
  //     .map(function (tag) {
  //       return tag[0] + "/" + tag[1];
  //     })
  //     .join(" ");

  //   // debugger;
  //   let nouns = chunker.chunk(tags, "[{ tag: NN|NNP|VBG }]");
  //   // console.log(nouns);
  //   let extractedNouns = [];
  //   let nouns2 = [...nouns.matchAll(/\{([^}]+)\}/g)];
  //   // console.log(nouns2);

  //   for (let n of nouns2) {
  //     // console.log(n);
  //     extractedNouns.push(n[1].match(/(.*?)\//)[1]);
  //   }

  //   // console.log("extractedNouns");
  //   // console.log(extractedNouns);

  //   let backgroundImageURL = `url("https://source.unsplash.com/random/?${extractedNouns.join()}")`;

  //   // console.log(backgroundImageURL);

  //   document.body.style.backgroundImage = backgroundImageURL;
  // }

function getIndividualWordTranslations() {
  Promise.all(
      rubyTexts.map((rubyText) =>
        fetch("https://translate.googleapis.com/translate_a/single?client=gtx&sl=tr&tl=en&dt=t&q=" + rubyText.word)
          .then((response) => response.json())
          .catch((e) => console.error(e))
      )
    )
    .then((texts) => {
      console.log(texts);
      for (let i = 0; i < texts.length; i++) {
        rubyTexts[i].translation = texts[i][0][0][0];
      }

      console.log(rubyTexts);
    })
}

  (async () => {
    const seed = makeid(4);
    console.log(seed);
    let a = await fetch(
      "https://tatoeba.elnu.com/?from=tur&orphans=no&sort=random&to=eng&trans_filter=limit&trans_to=eng&unapproved=no&limit=1&rand_seed=" +
        // "https://dev.tatoeba.org/en/api_v0/search?from=cmn&orphans=no&sort=random&to=eng&trans_filter=limit&trans_to=eng&unapproved=no&rand_seed=" +
        seed
      // WUHp sex
      // "h9ZP"
      // "8hQt"
      // "Po2n"
      // "0SbA"
      // "J73O"
      // "TxWJ"
      // "PSlA"
      // "JnkI"
    ).then((response) => response.json());

    // TODO: Cvy4

    let r = a.results[0];

    console.log(r);

    turkishText = r.text;

    

    let translation = r.translations.filter((a) => a.length > 0);
    translationText = translation[0][0].text;
    // translationText =
    //   "Maria and Natalia go shopping. They buy something for themselves.";
    // translationText = "I could hardly believe my ears when I heard the news.";
    // translationText = "I will tell him about it when he comes next time.";
    // console.log(translationText);

    document.getElementById("enText").innerText = translationText;

    // prep ruby texts for translations to come in
    rubyTexts = turkishText.split(" ").map(x => {
      return {
        word: x,
      };
    });
    
    // let extractedVerbs = [];
    // let verbs = chunker.chunk(tags, "[{ tag: VB|VBP }]");
    
    // for (let n of verbs.match(/\{([^}]+)\}/)) {
      //   extractedVerbs.push(n.match(/\{(.*)\//)[1]);
        // }
        // console.log("extractedVerbs");
        // console.log(extractedVerbs);
        
        
        getIndividualWordTranslations();
        
        SetBackgroundImage();
    }


    //   const fetchNames = async () => {
    //   try {
    //     const res = await Promise.all([
    //       axios.get("https://chinese-dictionary.azurewebsites.net/" + rubyText.chars),
    //       axios.get("./names-mid.json"),
    //       axios.get("./names-old.json")
    //     ]);
    //     const data = res.map((res) => res.data);
    //     console.log(data.flat());
    //   } catch {
    //     throw Error("Promise failed");
    //   }
    // };

    // for (let rubyText of rubyTexts) {
    //   if (rubyText.chars == "?" || rubyText.chars == "｡") {
    //     return;
    //   }
    //   let res = await (
    //     await axios.get(
    //       "https://chinese-dictionary.azurewebsites.net/" + rubyText.chars
    //     )
    //   ).data;
    //   // console.log(r);
    //   console.log(res);

    //   console.log(res.length && res[0].definitions);
    //   let r;

    //   // filter out surnames
    //   let res2 = res.filter((r) => !/[A-Z]/.test(r.pronunciation));
    //   res = res2.length ? res2 : res;

    //   // filter out variants?
    //   let res3 = res.filter((r) => !/variant/.test(r.definitions));
    //   console.log(res3);
    //   res = res3.length ? res3 : res;

    //   if (res.length) {
    //     let first = res[0].definitions.match(/^[^;]*/);
    //     if (first) {
    //       r = first[0];

    //       console.log(r);

    //       // you (informal, as opposed to courteous 您[nin2]) -> you
    //       let r2 = r.replace(/\([^()]*\)/g, "").trim();
    //       r = r2.length ? r2 : r;
    //       console.log("r");
    //       console.log(r);
    //     } else {
    //       // no ;
    //       r = res[0].definitions;

    //       // you (informal, as opposed to courteous 您[nin2]) -> you
    //       let r2 = r.replace(/\([^()]*\)/g, "").trim();
    //       r = r2.length ? r2 : r;
    //       console.log("r");
    //       console.log(r);
    //     }
    //   }

    //   rubyText.def = r;
    //   rubyTexts = rubyTexts;
    //   // console.log(rubyTexts);
    // }

    // if we are still missing a final ? we got to add it
    // if (rubyTexts[rubyTexts.length - 1].text[rubyTexts[rubyTexts.length - 1].text.length - 1] == "?"){
    //   if (rubyTexts[rubyTexts.length - 1].chars != "？"){
    //     rubyTexts = [...rubyTexts, console.]
    //   }
    // }

    // get translations for each word!!

    // for (let r of rubyTexts) {

    // }

    // let a = 0;
    // let t = chineseText;
    // for (let i = 0; i < syllableArray.length; i++) {
    //   if(t[a] == "，"){

    //   }
    //   else {
    //     const n = syllableArray[i];

    //     console.log(i, n, pinyinTextList[i], t.slice(a, a+n))
    //     a += n;
    //   }
    // }

    // for (let p of pinyinTextList) {
    //   const node = document.createElement("p");
    //   // const textnode = document.createTextNode(p);
    //   node.innerHTML = p;
    //   document.getElementById("pinyinTextDiv").appendChild(node);
    // }

    // console.log(rubyTexts);

  )();
  let msg = new SpeechSynthesisUtterance();

  /*
  function getIndividualWordTranslations() {
    // todo ignore ? in rubyText.chars
    // new more efficient way to get definitions with Promise.all!
    Promise.all(
      rubyTexts.map((rubyText) =>
        fetch("https://chinese-dictionary.azurewebsites.net/" + rubyText.chars)
          .then((response) => response.json())
          .catch((e) => console.error(e))
      )
    )
      // .then((res) => res.map((res) => (res ? res.data : [])))
      .then((texts) => {
        console.log("texts");
        console.log(texts);

        for (const [i, rubyText] of rubyTexts.entries()) {
          let res = texts[i];
          if (res.length == 0) {
            continue;
          }

          // filter out surnames
          let res2 = res.filter((r) => !/[A-Z]/.test(r.pronunciation));
          res = res2.length ? res2 : res;

          // filter out variants?
          let res3 = res.filter((r) => !/variant/.test(r.definitions));
          console.log(res3);
          res = res3.length ? res3 : res;

          // try to filter out definitions that use a different pronunciation
          let res4 = res.filter((r) => r.pronunciation == rubyText.text);
          console.log(res4);
          res = res4.length ? res4 : res;
          let r;

          // Todo: UUTa
          // 得
          // structural particle: used after a verb , linking it to following phrase indicating effect, degree, possibility etc

          if (res.length) {
            let first = res[0].definitions.match(/^[^;]/);
            if (first) {
              r = first[0];

              console.log(r);

              // you (informal, as opposed to courteous 您[nin2]) -> you
              let r2 = r.replace(/\([^()]*\)/g, "").trim();
              r = r2.length ? r2 : r;
              console.log("r");
              console.log(r);
            } else {
              // no ;
              r = res[0].definitions;

              // you (informal, as opposed to courteous 您[nin2]) -> you
              let r2 = r.replace(/\([^()]*\)/g, "").trim();
              r = r2.length ? r2 : r;
              console.log("r");
              console.log(r);
            }
          }

          rubyText.def = r;
          rubyTexts = rubyTexts;
        }
      });
  }
  */
</script>

<!-- 
<svelte:head>
  <script type="text/javascript" src="./rakutenma.js" charset="UTF-8"></script>
  <script
    type="text/javascript"
    src="./model_zh.js"
    charset="UTF-8"
    on:load={initZh}></script>
  <script type="text/javascript" src="./zh_chardic.js" charset="UTF-8"></script>
  <script type="text/javascript" src="./hanzenkaku.js" charset="UTF-8"></script>
</svelte:head> -->

<!-- <main>
  <div>
    <a href="https://vitejs.dev" target="_blank"> 
      <img src="/vite.svg" class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank"> 
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + Svelte</h1>

  <div class="card">
    <Counter />
  </div>

  <p>
    Check out <a href="https://github.com/sveltejs/kit#readme" target="_blank">SvelteKit</a>, the official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">
    Click on the Vite and Svelte logos to learn more
  </p>
</main> -->

{#if !turkishText}
<p>Generating Turkish Sentence</p>
{:else}

<div id="container">
  <!-- <p id="cmnText" />
  <div id="pinyinTextDiv" /> -->
  <!-- {#if pinyinTextList}
    {#each pinyinTextList as p}
      <p>{p}</p>
    {/each}
  {/if} -->
  <!-- <div id="ruby" /> -->
  <!-- <p id="turkishText">{turkishText}</p> -->

    <div id="ruby" style="display: flex; flex-wrap: wrap;  align-items: center; gap: calc(0.5rem + 0.45vw); justify-content: center;">
    <!-- {#if chineseText && (chineseText[chineseText.length - 1] == "。" || chineseText[chineseText.length - 1] == "？")}
      <span>&ensp;</span>
    {/if} -->
      {#if rubyTexts.length > 0}
        {#each rubyTexts as r}
            <div
              style="display: flex; flex-direction: column; align-items: center;"
            >
              <p
                style="font-size: calc(1rem + 0.45vw); word-wrap: break-word;width: fit-content; text-align: center; margin: 0; font-weight: normal"
              >
                {r.translation ?? " "}
    </p>

              <span style="font-size: calc(3rem + 0.45vw); font-weight: bold;">
                  {r.word}
              </span>

              
            </div>
        {/each}
      {:else}
        <small>Generating Chinese Sentence...</small>
      {/if}
  </div>
</div>
{/if}

<!-- <p id="pinyinText" /> -->
<hr style="margin: 1rem" />
<p id="enText" style="margin: 1.5rem;" />

<button
  on:click={() => {
    msg.text = turkishText;
    msg.lang = "tr-TR";
    msg.voice = voice;
    msg.rate = 0.8;
    window.speechSynthesis.speak(msg);
  }}>Speak sentence</button
>

<button
  style="margin-left: 0.5rem;"
  on:click={() => {
    navigator.clipboard.writeText(
      turkishText
    );
  }}>Copy Turkish Sentence</button
>

<br />
<br />

<!-- <style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style> -->
<style>
    #pinyinTextDiv {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    width: 100%;
    font-size: 1.8rem;
  }

  #pinyinTextDiv p {
    font-size: 1.8rem;
  }
  
  #turkishText {
    font-size: 2rem;
  }

  #container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 0.5rem;
  }
</style>
