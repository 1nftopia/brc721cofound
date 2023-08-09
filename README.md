# BRC721Cofound
|  -   | -  |
|  ----  | ----  |
| Name  | BRC721Cofound |
| Description  | A protocol enabling all Bitcoin users to collaboratively create a pixelated image on a public canvas, where each contribution is recorded on Bitcoin and represented by an inscription of the canvas's latest state. |
| Discussion-to | [https://bitcointalk.org/index.php?topic=5462453] |
| Category | Ordinals |


## Abstract
We propose a new standard for Ordinal NFTs that leverages Recursive Inscription, enabling all Bitcoin users to collaboratively and cost-effectively create a pixelated image on a public canvas, with each contribution being recorded on the Bitcoin blockchain and represented by an inscription reflecting the canvas's latest state.

## Motivation
Recursive Inscriptions allow us to generate a new inscription by referencing an existing one and making incremental modifications to it. This methodology is ideal for serial collaborative scenarios, such as collective coding, writing, and painting. In these scenarios, each contributor's effort can be tokenized through the creation of new inscriptions, which not only records these incremental efforts forever but also makes them tradable.

Leveraging this feature of Recursive Inscriptions, BRC721Cofound is designed as a standard for collectively creating pixelated images on Bitcoin. This allows anyone to freely express themselves on the public on-chain canvas and gain a tradable token in return. We believe this will be an exciting and Bitcoin-suited, fully on-chain game that has the potential to inspire crypto users from around the world, who may not know each other, to co-create magnificent digital artwork.

## Specifics
BRC721Cofound uses Recursive Inscriptions to record the entire process of everyone collectively creating a pixelated image. Each moment in this process is an inscription, which depicts what the jointly created pixelated image looks like at this moment. We call this inscription a Moment Inscription, which contains the pixels that are newly added or updated at this moment, and contains references to the previous Moment Inscription and the Code Inscription that processes the image changes between the two moments.

### Moment Inscription (NFT)

The Moment Inscription is defined as an HTML, which includes three parameters: newly added or updated pixels, the state of the canvas at the previous moment, and the code for handling pixel changes on the canvas. When ordinal browsers try to display this Moment Inscription, they will automatically execute the  main() function in the Code Inscription. It first loads the state of the canvas at the previous moment, then parses the newly added or updated pixels at the current moment and draws them on the canvas of the previous moment, finally obtaining the latest canvas state at the current moment.

```html
<!DOCTYPE html>
<html>
<head>
    <script>
        let p="[pixels such as 39630b3,38130b4,...]";
        let c="[last pixel inscription ID]";</script>
    <script src="/inscription/content/[code inscription ID]"></script>
</head>
</html>
```

### Code Inscription (NFT)

As we mentioned in the Moment Inscription, the primary role of the Code Inscription is to add the newly added or updated pixels to the canvas of the previous moment to create the current moment's canvas.

```javascript
async function getSnapshotData ( doc ) {
    let base64 = doc.querySelector( "meta[name=snapshot]" )?.content;

    if ( base64 == null ) return null;

    const img = new Image();

    img.src = base64;
    let load = false;
    img.onload = () => { load = true };
    while ( load != true ) { await sl( 1 ) } const canvas = document.createElement( 'canvas' );
    canvas.width = img.width;
    canvas.height = img.height;
    const context = canvas.getContext( '2d' );
    context.drawImage( img, 0, 0 );
    const imageData = context.getImageData( 0, 0, img.width, img.height );
    const pixels = imageData.data;
    let plist = [];
    for ( let x = 0;
        x < img.width;
        x++ ) {
            for ( let y = 0;
                y < img.height;
                y++ ) {
                    const pixelIndex = ( y * img.width + x ) * 4;
                const red = pixels[ pixelIndex ];
                const green = pixels[ pixelIndex + 1 ];
                const blue = pixels[ pixelIndex + 2 ];
                const alpha = pixels[ pixelIndex + 3 ];
                if ( alpha == 0xff ) {
                    const p = ( ( x << ( 4 * 5 ) ) + ( y << ( 4 * 3 ) ) + ( red >> 4 << ( 4 * 2 ) ) + ( green >> 4 << ( 4 * 1 ) ) + ( blue >> 4 << ( 4 * 0 ) ) ).toString( 16 ).padStart( 7, "0" );
                    plist.push( p )
                }
            }
    } return plist.join( ',' )
}

const css_b = "margin:0px; display: flex; justify-content: center; align-items: center; height: 100vh; background: #000";

let w = 256;
let h = 256;
let r = 1;

let sl = ( t ) => new Promise( ( r ) => setTimeout( r, t ) );
let td = ( s ) => ( { s: s, x: parseInt( s.substr( 0, 2 ), 16 ), y: parseInt( s.substr( 2, 2 ), 16 ), c: "#" + s.substr( 4, 3 ) } );
let tdl = ( l ) => l.split( ',' ).map( ( s ) => td( s ) );

let lh = async ( id ) => {
    try {
        let u = '/content/' + id;
        let x = new XMLHttpRequest();
        x.open( "GET", u, false );
        x.send( null );
        if ( x.status === 200 ) {
            let d = x.responseText;
            const doc = new DOMParser().parseFromString( d, 'text/html' );
            const pstr = await getSnapshotData( doc );
            if ( pstr != null ) return { p: pstr, c: null };
            let _p = d.match( /(?<=let p=")[^"]+/g );
            let _c = d.match( /(?<=let c=")[^"]+/g );
            let dd = { p: ( _p != null ? _p[ 0 ] : null ), c: ( _c != null ? _c[ 0 ] : null ) };
            return dd
        }
    } catch ( error ) { }
};

let dp = ( l, needCanvas = false ) => {
    const c = document.createElement( 'canvas' );
    const t = c.getContext( '2d' );
    c.width = w * r;
    c.height = h * r;
    l.reverse().forEach( p => {
        t.fillStyle = p.c;
        t.fillRect( p.x * r, p.y * r, r, r )
    } );
    if ( needCanvas ) return c;
    return c.toDataURL( 'image/png' )
};

const css_c = "max-height:100%; height: 100vw; image-rendering:pixelated";

window.addEventListener( 'DOMContentLoaded', async ( event ) => await main() );

let main = async () => {
    let l = ( p || window.p ).split( "," ).map( ( e ) => td( e ) );
    let _c = c || window.c;
    let b = document.body;
    b.style.cssText = css_b;
    while ( _c?.length > 0 ) {
        let g = await lh( _c );
        if ( g?.p.length > 0 ) { l.push( ...tdl( g.p ) ) } _c = g?.c || null
    } let cv = dp( l, true );
    cv.style.cssText = css_c;
    b.appendChild( cv );
    let h = document.head;
    let jl = h.querySelectorAll( "script" );
    jl.forEach( ( js ) => h.removeChild( js ) );
    let b64 = cv.toDataURL( 'image/png' );
    var m = document.createElement( "meta" );
    m.setAttribute( "name", "snapshot" );
    m.setAttribute( "content", b64 );
    h.appendChild( m );
    const ij = document.createElement( "script" );
    ij.text = `window.addEventListener('DOMContentLoaded',(event)=>{let img=new Image(); img.src=document.querySelector('meta[name=snapshot]')?.content; img.onload=()=>{let cv=document.querySelector('canvas'); let ctx=cv.getContext('2d'); ctx.drawImage(img,0,0)}});`;
    h.appendChild( ij )
};
```

## Rationale

Given the potentially large number of contributors to the pixelated image, rendering the lastest canvas state might require a deep recursion to load pixels drawn by everyone. This process, however, could lead to prolonged loading times in the inscription browser due to excessive recursion. 

To address this, the Code Inscription is designed to snapshot the latest canvas state once the current Moment Inscription is rendered. This snapshot is then stored within the current Moment Inscription's DOM tree. As a result, ordinal browsers can streamline the rendering process by caching the DOM tree of each rendered Moment Inscription, thereby reducing recursion levels. 

An example of a rendered Moment Inscription's DOM tree is provided below, which includes a 'snapshot' tag.

```html
<!DOCTYPE html>
<html>
<head>
    <meta name="snapshot" content="data:image/png;base64,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">
    <script>window.addEventListener( 'DOMContentLoaded', ( event ) => { let img = new Image(); img.src = document.querySelector( 'meta[name=snapshot]' )?.content; img.onload = () => { let cv = document.querySelector( 'canvas' ); let ctx = cv.getContext( '2d' ); ctx.drawImage( img, 0, 0 ) } } );</script>
</head>
<body style="margin: 0px; display: flex; justify-content: center; align-items: center; height: 100vh; background: rgb(0, 0, 0);">
    <canvas width="256" height="256" style="max-height: 100%; height: 100vw; image-rendering: pixelated;"></canvas>
</body>
</html>
```

## Backward Compatibility
All inscriptions produced by ERC721Cofound are standard Recursive Inscriptions that can be properly parsed by all ordinal browsers.

## Copyright
Copyright and related rights waived via [MIT](../LICENSE.md).



