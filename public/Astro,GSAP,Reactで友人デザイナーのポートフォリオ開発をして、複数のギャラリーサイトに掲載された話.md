---
title: 'Astro,GSAP,Reactで友人デザイナーのポートフォリオ開発をして、複数のギャラリーサイトに掲載された話'
tags:
  - TypeScript
  - ポートフォリオ
  - React
  - astro
  - gsap
private: false
updated_at: '2025-01-28T23:54:47+09:00'
id: cdcf623aa79efdde1ffb
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

先日、友人のデザイナー[佐藤 大輝](https://x.com/0521taiki1)さんから任せていただいていた、ポートフォリオの開発が完了し、リリースしました:clap::clap:

https://taikisato.com/

https://x.com/naoki_dev/status/1882015225513382350?s=46&t=MBlWwQiytXDLqSYi1xfVWQ

本記事では、依頼を受けた経緯やモチベーション、技術スタックや一部実装の方法をご紹介します！

## リポジトリ

https://github.com/INxST/st-portfolio

(スターいただけると嬉しいです...！)

## 各種ギャラリーサイトにも掲載いただきました:tada:

### [GOOD PORTFOLIO](https://goodportfol.io/)

https://goodportfol.io/detail/vXC6dnRI

https://x.com/goodportfol_io/status/1882205352260993464?s=46&t=MBlWwQiytXDLqSYi1xfVWQ

### [WebDesignClip](https://webdesignclip.com/)

https://webdesignclip.com/taikisato/

https://x.com/webdesignclip/status/1882322489310912885?s=46&t=MBlWwQiytXDLqSYi1xfVWQ

## 依頼を受けた経緯

[佐藤 大輝](https://x.com/0521taiki1)さんが私に依頼してくださったのは、

- なる早でポートフォリオをリニューアルしたい
- アニメーションをふんだんに盛り込みたい
  - (が、諸々考慮しつつ作成をすると時間がかかってしまう)

といった内容で、時間的な成約があるものの、その中でも可能な限り納得の行くものを作りたいという思いがあったようです。
時間的な成約と聞くとネガティブに捉えがちですが、そんな状況下で相談してくださっているのは、私の能力を買ってくれているとも捉えられるので、ポジティブに捉えることもできます。

▼相談いただいた際のやり取り(一部抜粋)

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3717182/63219e7e-fb17-2a58-9f8f-a0b94c0989fd.png" width="500" alt="キャプチャ">

「あなたとならより良いものを作れる」みたいなニュアンスですね。きっと。(自意識過剰かも)
この思いに惹かれて、依頼を受けることにしました。

## モチベーション

さて、経緯を読んでいただいた皆様の一部は、こんな疑問を抱いているかもしれません。

**「人様のポートフォリオ開発にモチベーションが湧くの？」**

はい。言わんとしていることはわかります。
ポートフォリオは作者自身を表現するものなので、第三者はモチベーションを見出しにくいですよね。あと、複数人で作成する実例もあまりなさそうな気がします...
が、私は

- [佐藤 大輝](https://x.com/0521taiki1)さんのデザインが好き
  - 特に、表現したい内容を的確に伝える手段としてデザインを駆使する姿勢が素敵だなと思っています
- 今回のポートフォリオは、アニメーションをふんだんに盛り込むということで、技術的な挑戦ができる
  - 私の興味関心がどちらかというとアプリケーションの実装の方向にあり、こういったサイトを作る機会は今後も少なそう
- (あわよくば)ギャラリーサイト掲載も狙えるかも？
  - 私はデザインの知見が乏しいので、一人では不可能ですが、[佐藤 大輝](https://x.com/0521taiki1)さんとであればもしかしたら！？

といった思いもあり、これらをモチベーションとして、開発を進めました。

## 技術スタック

### 前提

- [佐藤 大輝](https://x.com/0521taiki1)さん はHTML書けるので、更新は直接ソースを変更する形で良い
- アニメーションは結構させたい
- パフォーマンスにも気を使いたい

### 選定技術

前提を踏まえ、以下の技術を選定しました。

- Node.js + pnpm
- [Astro](https://astro.build/)
  - 今回の要件的にSSGで問題ないかつ、パフォーマンス面の考慮も楽にできるため
- [React](https://ja.react.dev/)
  - `.astro` だと実装が面倒な部分の補完用途で使用
- [TypeScript](https://www.typescriptlang.org/)
- [Prettier](https://prettier.io/)
  - フォーマッターとして採用
- [Tailwind CSS](https://tailwindcss.com/)
  - スタイリングに使用
- [Sass](https://sass-lang.com/)
  - アニメーション用スタイルなど、凝ったスタイリングに使用(極力使用は控えるようにはしました)
- [GSAP](https://greensock.com/gsap/)
  - アニメーション実装を楽にするため
- [Lenis](https://lenis.darkroom.engineering/)
  - 慣性スクロールを実装するために使用

ちなみに、大枠の選定については、以前開発した[Rowicyのサイト](https://www.rowicy.com/)の技術スタックを踏襲しました。

https://www.rowicy.com/blog/site-tech-stack/

## 実装紹介

個人的に気に入っている部分を3つピックアップしてご紹介します！

### 月の満ち欠けアニメーション

まず1つ目は、TOPページのヒーローセクションに実装している、下記のアニメーションです。

![moon.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3717182/70f1da02-ea53-067e-3e8b-76d59e7007cf.gif)

インパクトがある、目を引くアニメーションです...よね！？
ここはかなり苦戦した部分でもあって、はじめは疑似要素を動かして実装しようとしていましたが、どうしても自然原則に則った動きが再現できず、
最終的には、[公式のデモ](https://gsap.com/docs/v3/HelperFunctions/helpers/imageSequenceScrub/)を参考に、ペラペラ漫画のように画像を連続で描画することで月の満ち欠けを表現することにしました。

枚数はなんと、133枚です...！(画像は`moon_132.png`までですが、`moon_000.png`から始まっています)

![moon-list.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3717182/dde78b8a-9d67-6579-86dd-9f0745f67c7a.png)

実際のソースコードは以下です。

#### Hero Component

`canvas#moon` が月描画用要素です。
後述の `animation.ts` で、`imageSequence` を呼び出すことで、月の画像を連続で描画しています。

```html:index.astro
<section id="hero" class="relative">
  <div id="hero-container">
    <div
      class="overflow-y-hidden bg-mine-shaft-texture contents-full text-silver-chalice py-16 h-lvh flex flex-col justify-center"
    >
      <div
        class="flex flex-col lg:grid lg:grid-cols-3 lg:items-end justify-center"
      >
        <div
          class="relative inline-block overflow-hidden w-72 md:w-[50vw] lg:w-[34rem] h-72 md:h-[50vw] lg:h-[34rem] md:col-start-2 z-0 mx-auto lg:mx-0"
        >
          <canvas id="moon" class="w-full h-full" width="600" height="600"
          ></canvas>
        </div>

        <p class="font-serif-en ml-auto mt-4 lg:m-0 w-96 hidden md:block">
          I am Taiki Sato, a visual designer. <br />I am good at balancing
          logical and intuitive thinking, function and emotion, stillness and
          movement, and seeking and expressing the best visual expression that
          fulfills user needs and objectives.
        </p>
      </div>

      <div
        id="hero-video"
        class="absolute z-0 top-0 left-1/2 -translate-x-1/2
        w-screen h-full overflow-hidden opacity-50
        mix-blend-multiply transition-all duration-700
        is-loading [&.is-loading]:z-50"
      >
        <video
          poster={updatePath('/top/hero/shadow-image.png')}
          webkit-playsinline
          preload="auto"
          playsinline
          muted
          autoplay
          loop
          class="w-full h-full object-cover"
        >
          <source
            src={updatePath('/top/hero/shadow-movie.mp4')}
            type="video/mp4"
          />
        </video>
      </div>

      <p class="mt-14 font-serif-en md:text-xl">
        WEB DESIGN / GRAPHIC DESIGN / ART DIRECTION
      </p>

      <h1
        class="flex items-end justify-between md:justify-normal gap-4 md:gap-10 mt-3 md:mt-0"
      >
        <picture
          id="hero-logo"
          class="relative max-w-[19.6rem] md:max-w-[85rem] is-loading [&.is-loading]:z-50"
        >
          <source
            srcset={updatePath('/top/hero/fv-moving.png')}
            media="(min-width: 768px)"
          />
          <img
            src={updatePath('/top/hero/fv-moving-sp.png')}
            alt="TAIKI SATO"
          />
        </picture>

        <span class="inline-flex flex-col vertical-rl text-taupe-gray">
          <span
            class="font-serif-en text-sm md:text-lg leading-none whitespace-pre-line"
          >
            <span>(PORTFOLIO)</span>
          </span>
          <span
            class="font-medium text-[2rem] md:text-5xl leading-none whitespace-pre-line"
          >
            <span>作品集</span>
          </span>
        </span>
      </h1>
    </div>

    <div
      class="scroll-nav w-[1px] h-[17.5rem] bg-emperor absolute bottom-[5px] right-0 z-0 hidden md:block"
    >
      <div class="relative w-full h-full">
        <div
          class="scroll-nav__inside w-full h-10 bg-silver-chalice absolute top-0 left-0"
        >
        </div>
      </div>
    </div>
  </div>
</section>

<script>
  import loading from './loading';
  import animation from './animation';
  loading();
  animation();
</script>
```

#### animation.ts

こちらのscriptで、以下を行っています。

- 描画する画像URLの配列を作成
- `imageSequence` を呼び出し、月の画像を描画
  - 引数には、`urls`(画像URL配列) と `canvas`(canvasセレクタ) 、`scrollTrigger`(GSAPのScrollTrigger) を渡しています
- `gsap` を使用して、スクロールに合わせてコンテナをアニメーション

```ts:animation.ts
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import imageSequence from './imageSequence';

const animation = () => {
  gsap.registerPlugin(ScrollTrigger);

  const hero = document.getElementById('hero');
  const height = hero?.clientHeight;
  const container = document.getElementById('hero-container');

  const frameCount = 133;
  const urls = new Array(frameCount)
    .fill(null)
    .map(
      (_o, i) =>
        `${location.href}/top/moon/moon_${i.toString().padStart(3, '0')}.png`
    );

  imageSequence({
    urls,
    canvas: '#moon',
    scrollTrigger: {
      start: 0,
      end: () => `+=${height! * 0.8}`,
      scrub: true,
    },
  });

  gsap
    .timeline({
      scrollTrigger: {
        trigger: hero,
        start: 'top top',
        end: () => `+=${height}`,
        scrub: true,
      },
    })
    .to(
      container,
      { transform: 'translate3d(0, 20rem, 0)', ease: 'Power4.out' },
      '<'
    );
};

export default animation;
```

#### imageSequence.ts

`animation.ts` で引数として受け取った `urls` を元に、月の画像を描画し、受け取った `scrollTrigger` に合わせて描画を更新しています。
(一応、他にも引数として設定できるようにしていますが、今回は使っていません)

```ts:imageSequence.ts
import gsap from 'gsap';

type ImageSequenceConfig = {
  urls: string[];
  canvas: HTMLCanvasElement | string;
  clear?: boolean;
  scrollTrigger?: ScrollTrigger.StaticVars;
  paused?: boolean;
  fps?: number;
  onUpdate?: (index: number, image: HTMLImageElement) => void;
};

const imageSequence = (
  config: ImageSequenceConfig
): gsap.core.Tween | undefined => {
  let playhead = { frame: 0 },
    canvas =
      typeof config.canvas === 'string'
        ? document.querySelector<HTMLCanvasElement>(config.canvas)
        : config.canvas;

  if (!canvas) {
    console.warn('Canvas not defined');
    return;
  }

  let ctx = canvas.getContext('2d'),
    curFrame = -1,
    onUpdate = config.onUpdate,
    images: HTMLImageElement[];

  const updateImage = () => {
    let frame = Math.round(playhead.frame);
    if (frame !== curFrame && ctx) {
      if (config.clear) {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
      }
      ctx.drawImage(images[frame], 0, 0);
      curFrame = frame;
      if (onUpdate) {
        onUpdate(frame, images[frame]);
      }
    }
  };

  images = config.urls.map((url, i) => {
    let img = new Image();
    img.src = url;
    if (i === 0) {
      img.onload = updateImage;
    }
    return img;
  });

  return gsap.to(playhead, {
    frame: images.length - 1,
    ease: 'none',
    onUpdate: updateImage,
    duration: images.length / (config.fps || 30),
    paused: !!config.paused,
    scrollTrigger: config.scrollTrigger,
  });
};

export default imageSequence;
```

### テキスト回転アニメーション

2つ目は、縦書きテキストリンクhover時の回転アニメーションです。

![rotate-text.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3717182/8f9fbefc-9352-e542-ea0b-01e0a6dc787b.gif)

#### HTML

- `.ts-text-link` が script で処理するための class
- `data-vertical` が文字方向を指定する属性

です。
他はリンク先とスタイリング用の属性を付与した、シンプルな a タグです。

```html
<a
  href="/about/"
  class="ts-text-link text-2xl md:text-lg vertical-rl"
  data-vertical="rl"
>
  私について
</a>
```

### `.ts-text-link` を アニメーション用の構造に書き換え

script で、アニメーション用の構造に書き換えます。
一文字ずつ分割して複製し、span タグで囲み、それぞれに遅延を設定しています。

:::note warn
本来は script でのDOM操作ではなく、直接 HTML に記述するなどの方法が望ましいです。
今回は更新時の可読性を優先して、 script でのDOM操作を採用しています。
:::

```typescript:hoverTextLink.ts
const hoverTextLink = () => {
  const links = document.querySelectorAll<HTMLElement>('.ts-text-link');
  const delay = 50;
  links.forEach(link => {
    // 一文字ずつ分割して複製し、spanタグで囲む
    const text = link.textContent;
    const splitText = text?.split('');
    // 空文字を除外
    const filteredText = splitText?.filter(char => char !== ' ');

    const wrappedText = filteredText?.map((char, i) => {
      return `<span style="transition-delay:${i * delay}ms;">${char}</span>`;
    });

    // オリジナルは残して、spanタグで囲んだテキストを挿入
    link.innerHTML = `<span class="ts-text-link__original">${text}</span>`;
    link.innerHTML += `<span class="ts-text-link__clone" aria-hidden="true">${wrappedText?.join('')}</span>`;
    link.innerHTML += `<span class="ts-text-link__clone" aria-hidden="true">${wrappedText?.join('')}</span>`;
  });
};

export default hoverTextLink;
```

書き換え後は下記のようになります。

```html
<a href="/about/"
  class="ts-text-link text-2xl md:text-lg vertical-rl"
  data-vertical="rl"
>
  <span class="ts-text-link__original"> 私について </span>
  <span class="ts-text-link__clone" aria-hidden="true">
    <span style="transition-delay:0ms;">私</span>
    <span style="transition-delay:50ms;">に</span>
    <span style="transition-delay:100ms;">つ</span>
    <span style="transition-delay:150ms;">い</span>
    <span style="transition-delay:200ms;">て</span>
  </span>
  <span class="ts-text-link__clone" aria-hidden="true">
    <span style="transition-delay:0ms;">私</span>
    <span style="transition-delay:50ms;">に</span>
    <span style="transition-delay:100ms;">つ</span>
    <span style="transition-delay:150ms;">い</span>
    <span style="transition-delay:200ms;">て</span>
  </span>
</a>
```

同じ文字列が複数ありますが、

- `.ts-text-link__original` は元のテキスト (常時非表示、スクリーンリーダー読み上げ用)
- 1つ目の `.ts-text-link__clone` は 未hover時のみ表示
- 2つ目の `.ts-text-link__clone` は hover時のみ表示

としています。

#### スタイリング

前述の役割に合わせて、スタイリングをしています。
(`data-vertical='rl'` が指定されている場合は、横書き用のスタイルを適用しています。)

```scss
.ts-text-link {
  $this: &;
  position: relative;

  * {
    transition-duration: 0.8s;
    transition-property: opacity, visibility, transform, top, left;
    transition-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
  }

  &__original {
    opacity: 0;
    padding: 0 0.1em;
  }

  &__clone {
    position: absolute;
    top: 0;
    left: 0;
    display: flex;
  }

  &:not([data-vertical='rl']) {
    display: inline-flex;
    overflow-y: hidden;

    #{$this}__clone {
      + #{$this}__clone {
        span {
          transform: translateY(120%);
        }
      }
    }

    &:hover {
      #{$this}__clone {
        span {
          transform: translateY(-120%);
        }

        + #{$this}__clone {
          span {
            transform: translateY(0);
          }
        }
      }
    }
  }

  &[data-vertical='rl'] {
    overflow-x: hidden;

    #{$this}__clone {
      + #{$this}__clone {
        span {
          transform: translateX(120%);
        }
      }
    }

    &:hover {
      #{$this}__clone {
        span {
          transform: translateX(-120%);
        }

        + #{$this}__clone {
          span {
            transform: translateX(0);
          }
        }
      }
    }
  }
}
```

### 画像 パララックススクロールアニメーション

最後は、下記の画像パララックススクロールアニメーションです。

![parallax-image.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3717182/dcf12bfe-68a1-d26d-34ba-972271b568ec.gif)

#### Section Component

画像情報オブジェクト `parallaxGalleryItems` をループして `Item` コンポーネントを描画しています。
`animation.ts` がアニメーション用の script です。

```html:index.astro
---
import TopTitle from '../TopTitle.astro';
import More from '../More.astro';
import parallaxGalleryItems from '@/data/parallaxGalleryItems';
import Item from './Item.astro';
---

<section id="parallax-gallery">
  <div
    class="contents-full text-pampas relative py-[4.6rem] md:py-44 h-screen overflow-hidden"
  >
    <div class="bg-mine-shaft-texture absolute top-0 left-0 w-full h-full">
    </div>

    <div class="flex justify-between">
      <ul class="flex-1 flex flex-col items-start">
        {parallaxGalleryItems.left.map(item => <Item item={item} />)}
      </ul>

      <ul class="flex-1 flex flex-col items-end">
        {parallaxGalleryItems.right.map(item => <Item item={item} />)}
      </ul>
    </div>

    <div
      class="flex flex-col justify-center items-center w-screen h-screen
      absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 z-20"
    >
      <div class="text-taupe-gray">
        <TopTitle
          sub="（私について）"
          title={['これまでのこと', '嗜好と思考の話']}
          href="/about"
        />
      </div>

      <div
        class="ts-focus-in data-[active='true']:animate-text-focus-in mt-16 md:mt-[5.5rem]"
      >
        <More color="bright" href="/about" />
      </div>
    </div>
  </div>
</section>

<script>
  import animation from './animation';
  animation();
</script>
```

#### parallaxGalleryItems.ts

##### 型

```typescript:ParallaxGalleryItem.ts
type ParallaxGalleryItem = {
  src: string;
  alt: string;
  class?: string;
};

export type { ParallaxGalleryItem };
```

##### 画像情報オブジェクト

画像によって、マージン感やサイズなどが異なるため、それぞれの画像情報をオブジェクトとして管理しています。

```ts:parallaxGalleryItems.ts
import type { ParallaxGalleryItem } from '@/types/ParallaxGalleryItem';

type ParallaxGalleryItems = {
  left: ParallaxGalleryItem[];
  right: ParallaxGalleryItem[];
};

const parallaxGalleryItems: ParallaxGalleryItems = {
  left: [
    {
      src: '/top/about/left-01.jpg',
      alt: 'left-01',
      class: 'w-46 md:w-[27rem] md:ml-[10rem]',
    },
    // ...
  ],
  right: [
    {
      src: '/top/about/right-01.jpg',
      alt: 'right-01',
      class: 'w-37 md:w-[26rem] mt-20 md:mt-[13rem] -mr-10 md:mr-[6rem]',
    },
    // ...
  ],
};

export default parallaxGalleryItems;
```

#### Item Component

画像単体のコンポーネントです。
(styleタグは 画像の色味が戻っていくアニメーション用です)

```html:Item.astro
---
import updatePath from '@/libs/updatePath';
import cn from '@/libs/cn';
import type { ParallaxGalleryItem } from '@/types/ParallaxGalleryItem';
interface Props {
  item: ParallaxGalleryItem;
}
const { item } = Astro.props;
const itemClass = cn(
  'ts-parallax-gallery-image relative overflow-hidden',
  item.class
);
---

<li class={itemClass}>
  <picture class="w-full block mix-blend-multiply">
    <img src={updatePath(item.src)} alt={item.alt} class="w-full" />
  </picture>
  <div
    class="bg-layer absolute -bottom-1/2 left-0 w-full h-[200%] bg-taupe-gray -z-10"
  >
  </div>
</li>

<style lang="scss">
  .ts-parallax-gallery-image {
    transition: filter 0.5s ease;

    img,
    .bg-layer {
      transition: all 1s ease;
    }

    @for $i from 1 through 8 {
      &:nth-child(#{$i}) {
        .bg-layer {
          transform: skewY(#{if($i % 2 == 0, 10, -10)}deg);
        }
      }
    }

    &:not(.is-active) {
      img {
        filter: grayscale(100%);
      }
    }

    &.is-active {
      img {
        filter: grayscale(0);
      }

      .bg-layer {
        transform: translateY(100%);
        opacity: 0;
      }
    }
  }
</style>
```

#### パララックススクロールアニメーション

`animation.ts` で、画像のスクロールに合わせてアニメーションを実装しています。
尚、より立体感を出すために、各画像のスピードを変えています。

```ts:animation.ts
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

const animation = () => {
  gsap.registerPlugin(ScrollTrigger);
  const selectorImage = '.ts-parallax-gallery-image';
  const yPercents = [500, 1000, 1500];
  const scrubs = [1, 2, 3];
  const target = document.getElementById('parallax-gallery');
  const images = gsap.utils.toArray<HTMLElement>(selectorImage);

  gsap.timeline({
    scrollTrigger: {
      trigger: target,
      start: 'top top',
      end: () => `+=${target?.clientHeight! - 250}`,
      scrub: true,
      pin: true,
    },
  });

  images.forEach((image, i) => {
    // iの値によってyPercentとscrubの値を変える
    const yPercent = yPercents[i % yPercents.length];
    const scrub = scrubs[i % scrubs.length];
    gsap.set(image, { zIndex: yPercent / 100, position: 'relative' });
    gsap.fromTo(
      image,
      {
        yPercent: yPercent,
      },
      {
        yPercent: `-${yPercent}`,
        ease: 'none',
        scrollTrigger: {
          trigger: target,
          start: 'top bottom',
          end: 'bottom top',
          scrub: scrub,
        },
      }
    );
  });

  window.addEventListener('scroll', () => {
    const scroll = window.scrollY;
    const windowHeight = window.innerHeight;

    images.forEach(image => {
      const targetPos = image.getBoundingClientRect().top + scroll;

      if (scroll > targetPos - windowHeight / 2) {
        image.classList.add('is-active');
      }
    });
  });
};

export default animation;
```

## 最後に

今回のポートフォリオ開発は、技術的な挑戦ができるとともに、[佐藤 大輝](https://x.com/0521taiki1)さんとのコラボレーションを楽しむことができました。
デザイナーとして活躍している彼とだからこそ作り上げることができた、最高の作品になったと自負しています！
ポートフォリオの共同開発、一人で作るのとはまた違った良さがあり、オススメなので、機会があればぜひ挑戦してみてください！

最後まで読んでいただき、ありがとうございました:bow:
