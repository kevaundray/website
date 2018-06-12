# Test vectors for `ristretto255`

```rust
// The test vectors below also have code to test them against the
// curve25519_dalek implementation.

extern crate sha2;
extern crate hex;
extern crate curve25519_dalek;

# fn main() {
// The following are the byte encodings of small multiples 
//     [0]B, [1]B, ..., [15]B
// of the basepoint, represented as hex strings.

let encodings_of_small_multiples = [
    // This is the identity point
    "0000000000000000000000000000000000000000000000000000000000000000",
    // This is the basepoint
    "e2f2ae0a6abc4e71a884a961c500515f58e30b6aa582dd8db6a65945e08d2d76",
    // These are small multiples of the basepoint
    "6a493210f7499cd17fecb510ae0cea23a110e8d5b901f8acadd3095c73a3b919",
    "94741f5d5d52755ece4f23f044ee27d5d1ea1e2bd196b462166b16152a9d0259",
    "da80862773358b466ffadfe0b3293ab3d9fd53c5ea6c955358f568322daf6a57",
    "e882b131016b52c1d3337080187cf768423efccbb517bb495ab812c4160ff44e",
    "f64746d3c92b13050ed8d80236a7f0007c3b3f962f5ba793d19a601ebb1df403",
    "44f53520926ec81fbd5a387845beb7df85a96a24ece18738bdcfa6a7822a176d",
    "903293d8f2287ebe10e2374dc1a53e0bc887e592699f02d077d5263cdd55601c",
    "02622ace8f7303a31cafc63f8fc48fdc16e1c8c8d234b2f0d6685282a9076031",
    "20706fd788b2720a1ed2a5dad4952b01f413bcf0e7564de8cdc816689e2db95f",
    "bce83f8ba5dd2fa572864c24ba1810f9522bc6004afe95877ac73241cafdab42",
    "e4549ee16b9aa03099ca208c67adafcafa4c3f3e4e5303de6026e3ca8ff84460",
    "aa52e000df2e16f55fb1032fc33bc42742dad6bd5a8fc0be0167436c5948501f",
    "46376b80f409b29dc2b5f6f0c52591990896e5716f41477cd30085ab7f10301e",
    "e0c418f7c8d9c4cdd7395b93ea124f3ad99021bb681dfc3302a9d99a2e53e64e",
];

// Test the encodings of small multiples

use curve25519_dalek::constants;
use curve25519_dalek::traits::Identity;
use curve25519_dalek::ristretto::RistrettoPoint;

let B = &constants::RISTRETTO_BASEPOINT_POINT;
let mut P = RistrettoPoint::identity();
for i in 0..16 {
    assert_eq!(
        hex::encode(P.compress().as_bytes()),
        encodings_of_small_multiples[i],
    );
    P = P + B;
}

// The following are invalid encodings, which should all be rejected.

let bad_encodings = [
    // These are all bad because they're non-canonical field encodings.
    "00ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
    "ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff7f",
    "f3ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff7f",
    "edffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff7f",
    // These are all bad because they're negative field elements.
    "0100000000000000000000000000000000000000000000000000000000000000",
    "01ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff7f",
    // These are all bad because they give a nonsquare x^2.
    "26948d35ca62e643e26a83177332e6b6afeb9d08e4268b650f1f5bbd8d81d371",
    "4eac077a713c57b4f4397629a4145982c661f48044dd3f96427d40b147d9742f",
    "de6a7b00deadc788eb6b6c8d20c0ae96c2f2019078fa604fee5b87d6e989ad7b",
    "bcab477be20861e01e4a0e295284146a510150d9817763caf1a6f4b422d67042",
    "2a292df7e32cababbd9de088d1d1abec9fc0440f637ed2fba145094dc14bea08",
    "f4a9e534fc0d216c44b218fa0c42d99635a0127ee2e53c712f70609649fdff22",
    "8268436f8c4126196cf64b3c7ddbda90746a378625f9813dd9b8457077256731",
    "2810e5cbc2cc4d4eece54f61c6f69758e289aa7ab440b3cbeaa21995c2f4232b",
    // More bad encodings... these should hit each of the 4 checks.
];

// Test that all of the bad encodings are rejected

use curve25519_dalek::ristretto::CompressedRistretto;

let mut bad_bytes = [0u8; 32];
for bad_encoding in &bad_encodings {
    bad_bytes.copy_from_slice(&hex::decode(bad_encoding).unwrap());
    assert!(CompressedRistretto(bad_bytes).decompress().is_none());
}

// The following are a list of UTF-8 encoded strings, and the byte
// encodings of performing hash-to-point with SHA-512.

let labels = [
    "Ristretto is traditionally a short shot of espresso coffee",
    "made with the normal amount of ground coffee but extracted with",
    "about half the amount of water in the same amount of time",
    "by using a finer grind.", 
    "This produces a concentrated shot of coffee per volume.",
    "Just pulling a normal shot short will produce a weaker shot",
    "and is not a Ristretto as some believe.",
];

let encoded_hash_to_points = [
    "5c827272de19f868edc97deac8a9b12dea4604807463b59b040d463b8fa7f77f",
    "88c01ea47c9a049578adced99d439d0f09a847595c546d51c6082b891f9dbc65",
    "722b48912cd619ad2d542e78293f11053f8d18dcae456c7c7c98923d5eda844d",
    "fafc78dec2fd2565d8592320fe0632ec288ffd3acd55314e619fdb9884dabe1c",
    "8ae1f2311f26c03f14bcc192c0dd7589f679019068e6c83907e01735810a817c",
    "6a152dcc33dfda4554e4999b439fe734f11021502825f14a1745e44dd023423b",
    "b2757b09bc0ab160b2965e8e6385b3238e04665159b65ba26f1526303ebda807",
];

// Test the encodings above.

use sha2::{Sha512, Digest};

for i in 0..7 {
    let point = RistrettoPoint::hash_from_bytes::<Sha512>(labels[i].as_bytes());
    assert_eq!(
        hex::encode(point.compress().as_bytes()),
        encoded_hash_to_points[i],
    );
}
# }
```