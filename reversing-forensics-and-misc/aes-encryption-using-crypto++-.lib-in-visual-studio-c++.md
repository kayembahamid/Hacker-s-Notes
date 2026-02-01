# AES Encryption Using Crypto++ .lib in Visual Studio C++

## AES Encryption Using Crypto++ .lib in Visual Studio C++

This is a quick note showing how to compile, link and include a [Crypto++](https://www.cryptopp.com) static library (cryptlib.lib), compile and execute a sample code that uses AES CBC to encrypt and decrypt some string data.

### Compiling cryptlib.lib

Open the crypto++ solution file cryptest.sln:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lp5089xAkItAl1DHIUk%2F-Lp53ZmPcH8jdKoM0QPH%2Fimage.png?alt=media\&token=59304336-6802-4476-adf2-6f6322d25ac4)

Change cryptlib project runtime library to `Multi-threaded` and change configuration to `Release` `x64`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lp5089xAkItAl1DHIUk%2F-Lp50u5z7jKoUSgEA6k4%2Fimage.png?alt=media\&token=d228a4d2-2540-472c-95d6-57d4eb63935b)

Build cryptlib project. It will spit out a cryptlib.lib static library:

```
C:\Users\mantvydas\Desktop\cryptopp\x64\Output\Release\cryptlib.lib
```

### Including cryptlib.lib in a Project

Create a new VS project and include cryptlib.lib that you've just compiled:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lp5089xAkItAl1DHIUk%2F-Lp51CnxtMx1fmLM7RlN%2Fimage.png?alt=media\&token=0936dd24-003c-4a77-9ff2-20fb8730ea50)

Change project's runtime library to Multi-threaded - it has to use the same runtime library as cryptlib.lib:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lp5089xAkItAl1DHIUk%2F-Lp51Ok23o-QjUlsLX-V%2Fimage.png?alt=media\&token=35f6cd47-a5dc-46bc-8f14-d31db510fc67)

Copy over all the header files from the crypto++ project to your project's folder like so:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lp5089xAkItAl1DHIUk%2F-Lp51Urvq46c8YtTod9j%2Fimage.png?alt=media\&token=e9f9d207-07a6-47f9-aa14-8d07c5a4c29d)

Include those headers in the project by adding the folder to `Include Directories` list:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lp5089xAkItAl1DHIUk%2F-Lp51h31fBjRSqpEun9j%2Fimage.png?alt=media\&token=16b83784-7048-4a0c-8919-8f1fee9f6308)

Copy over the below sample code to your main .cpp file and compile:

{% code title="crypto.cpp" %}
```cpp
// code copy pasted from here https://www.cryptopp.com/w/images/b/bd/AES-CBC-Filter.zip
// crypto.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include "pch.h"
#include <iostream>
#include "aes.h"
#include <Windows.h>

#include "osrng.h"
using CryptoPP::AutoSeededRandomPool;

#include <iostream>
using std::cout;
using std::cerr;
using std::endl;

#include <string>
using std::string;

#include <cstdlib>
using std::exit;

#include "cryptlib.h"
using CryptoPP::Exception;

#include "hex.h"
using CryptoPP::HexEncoder;
using CryptoPP::HexDecoder;

#include "filters.h"
using CryptoPP::StringSink;
using CryptoPP::StringSource;
using CryptoPP::StreamTransformationFilter;

#include "aes.h"
using CryptoPP::AES;

#include "ccm.h"
using CryptoPP::CBC_Mode;

#include "assert.h"

int main(int argc, char* argv[])
{
	AutoSeededRandomPool prng;

	byte key[AES::DEFAULT_KEYLENGTH];
	prng.GenerateBlock(key, sizeof(key));

	byte iv[AES::BLOCKSIZE];
	prng.GenerateBlock(iv, sizeof(iv));

	string plain = "CBC Mode Test";
	string cipher, encoded, recovered;

	/*********************************\
	\*********************************/

	// Pretty print key
	encoded.clear();
	StringSource(key, sizeof(key), true,
		new HexEncoder(
			new StringSink(encoded)
		) // HexEncoder
	); // StringSource
	cout << "key: " << encoded << endl;

	// Pretty print iv
	encoded.clear();
	StringSource(iv, sizeof(iv), true,
		new HexEncoder(
			new StringSink(encoded)
		) // HexEncoder
	); // StringSource
	cout << "iv: " << encoded << endl;

	/*********************************\
	\*********************************/

	try
	{
		cout << "plain text: " << plain << endl;

		CBC_Mode< AES >::Encryption e;
		e.SetKeyWithIV(key, sizeof(key), iv);

		// The StreamTransformationFilter removes
		//  padding as required.
		StringSource s(plain, true,
			new StreamTransformationFilter(e,
				new StringSink(cipher)
			) // StreamTransformationFilter
		); // StringSource

#if 0
		StreamTransformationFilter filter(e);
		filter.Put((const byte*)plain.data(), plain.size());
		filter.MessageEnd();

		const size_t ret = filter.MaxRetrievable();
		cipher.resize(ret);
		filter.Get((byte*)cipher.data(), cipher.size());
#endif
	}
	catch (const CryptoPP::Exception& e)
	{
		cerr << e.what() << endl;
		exit(1);
	}

	/*********************************\
	\*********************************/

	// Pretty print
	encoded.clear();
	StringSource(cipher, true,
		new HexEncoder(
			new StringSink(encoded)
		) // HexEncoder
	); // StringSource
	cout << "cipher text: " << encoded << endl;

	/*********************************\
	\*********************************/

	try
	{
		CBC_Mode< AES >::Decryption d;
		d.SetKeyWithIV(key, sizeof(key), iv);

		// The StreamTransformationFilter removes
		//  padding as required.
		StringSource s(cipher, true,
			new StreamTransformationFilter(d,
				new StringSink(recovered)
			) // StreamTransformationFilter
		); // StringSource

#if 0
		StreamTransformationFilter filter(d);
		filter.Put((const byte*)cipher.data(), cipher.size());
		filter.MessageEnd();

		const size_t ret = filter.MaxRetrievable();
		recovered.resize(ret);
		filter.Get((byte*)recovered.data(), recovered.size());
#endif

		cout << "recovered text: " << recovered << endl;
	}
	catch (const CryptoPP::Exception& e)
	{
		cerr << e.what() << endl;
		exit(1);
	}

	/*********************************\
	\*********************************/

	return 0;
}
```
{% endcode %}

Success:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lp5089xAkItAl1DHIUk%2F-Lp51tTpBho3s54UHHJ2%2Fimage.png?alt=media\&token=362050e8-45fa-4227-af07-4fb94bed9c7f)

### References

{% embed url="https://www.cryptopp.com/w/images/b/bd/AES-CBC-Filter.zip" %}

{% embed url="https://stackoverflow.com/questions/36000317/link-errors-using-cryptopp-on-vs2012-static-library-console-application-and-clr" %}

{% embed url="https://www.cryptopp.com/" %}
