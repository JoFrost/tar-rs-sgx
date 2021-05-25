# tar-rs

[Documentation](https://docs.rs/tar)

A tar archive reading/writing library for Rustm, adapted for Rust SGX, for the enclave part.

[Original github repo](https://github.com/alexcrichton/tar-rs)
[Rust SGX SDK](https://github.com/apache/incubator-teaclave-sgx-sdk)

```toml
# Cargo.toml
[dependencies]
tar = "0.4"
```

## Reading an archive (host part)

```rust,no_run
extern crate tar;

use std::io::prelude::*;
use std::fs::File;

fn main() -> std::io::Result<()> {
    let file = File::open("foo.tar").unwrap();
    let mut buffer = vec![0; metadata.len() as usize];
    file.read(&mut buffer).expect("buffer overflow");
    let mut array = &buffer[..];
    let mut retval = sgx_status_t::SGX_SUCCESS;

    let result = unsafe {
        read_tar(enclave.geteid(),
                      &mut retval,
                      array.as_ptr() as * const u8,
                      array.len())
    };

    match result {
        sgx_status_t::SGX_SUCCESS => {},
        _ => {
            println!("[-] ECALL Enclave Failed {}!", result.as_str());
            let error = Error::new(ErrorKind::Other, "ECALL failed");
            return Err(error);
        }
    }
    Ok(())
}

```

## Reading an archive (enclave part)

```rust,no_run
#![crate_name = "tar_test"]
#![crate_type = "staticlib"]

extern crate tar;

#![cfg_attr(not(target_env = "sgx"), no_std)]
#![cfg_attr(target_env = "sgx", feature(rustc_private))]

extern crate sgx_types;
#[cfg(not(target_env = "sgx"))]
#[macro_use]
extern crate sgx_tstd as std;

use std::io::prelude::*;
use std::io::Read;
use tar::Archive;

pub extern "C" fn read_tar(tar_ptr: *const u8, tar_len: usize) -> sgx_status_t 
{
    let mut tar_slice = unsafe { slice::from_raw_parts(tar_ptr, tar_len) };
    let mut a = Archive::new(tar_slice);

    for entry in a.entries() {
            let mut entry = entry?;
            let path = entry.path()?.to_path_buf();
    }
    sgx_status_t::SGX_SUCCESS
}

```

# License

This project is licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
   http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or
   http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this project by you, as defined in the Apache-2.0 license,
shall be dual licensed as above, without any additional terms or conditions.
