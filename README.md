## Belajar Docker Pemula

#### Persiapan
1. Download dan Install Docker dulu kalo belum, cek aja di link ini https://www.docker.com/get-started
2. Kalo ke install bisa cek gini ya:
    ```dockerfile
    docker info
   
   docker version
    ```
3. Selesai

#### Kenalan
Docker itu sebuah **container** bukan sebuah **virtual machine** jadi kalo udah ada docker kita gak perlu lagi install operating sistem.
Docker sendiri udah menggunakan sistem operasi linux.

Pertanyaan nya, kenapa sih harus belajar Docker?

Sebenernya kalo yang belum mulai terjun banget ke dunia perkodingan, mungkin di rasa belum terlalu perlu ya kita tau docker,
tapi kalo udah terjun jauh tau docker itu bisa di bilang wajib wkkw. Kenapa wajib, ya karena buat development sama production sangat bantu banget.

Contohnya gini deh, kita kan kalo buat aplikasi pasti pake db dong? nah dengan docker kita bisa tu install banyak dataabse server di dalamnnya
tanpa ganggu komputer kita, gampang nya gitu hhe.


#### Arsitektur Docker
![alt text](https://docs.docker.com/engine/images/architecture.svg)

#### Memulai Docker

##### Docker Registry
Untuk menyimpan docker image, beberapa yang ada di docker registry:
1. Docker Hub (Bawaan docker) -> https://hub.docker.com/
2. Google Cloud
3. AWS Elastic

##### Docker Image
Hasil dari distribusi file (package/bundle), nah image ini yang akan kita deploy ke docker registry dan sudah siap di gunakan.
Untuk melihat image yang tersedia silahkan buka di docker hub.

#### Docker Container
Container adalah, image yang kita running.
Dalam docker dapat menjalankan banyak container.


#### Pull Image
Kita lihat daftar image dulu di local / Laptop kita, caranya ketik ini:
```
docker images
```

Disini saya mencoba untuk mendownload image mongo, caranya adalah ketik ini:
```
docker pull mongo
```
Secara default akan mendownload image versi/tags terakhir, tetapi jika kita ingin mendownload tag tertentu tinggal ketik ini aja:
```
docker pull mongo:4.4.2
```

#### Membuat Container
1. Setelah image mongo tadi selesai didownload, sekarang kita buat sebuah container
dengan cara:
    Expose port ke luar dengan port 8080
    ```
    docker container create --name mongoserver -p 8080:27017 mongo:4.4.2
    ```
   Expose port ke luar dengan port 8181
   ```
    docker container create --name mongoserver2 -p 8181:27017 mongo:4.4.2
    ```
2. Menjalankan container :
    ```
   docker container start mongoserver mongserver2
   ```
3. Uji coba menggunakan aplikasi Studio 3T atau sql client lainnya yang sesuai.

#### Menghapus Container
Untuk menghapus silahkan stop dulu containernya:
```
docker container stop namacontainer
```
baru ketik ini untuk menghapusnya:
```
docker container rm namaconatiner
```

#### Menghapus Image
Untuk menghapus silahkan cek dulu containernya masih di pakai atau tidak, kalo masih running silahkan stop dulu. Kemudian hapus container nya.
Lalu lakukan ini:
```
docker image rm namaconatiner
```

#### Membuat Image Dengan Dockerfile
Silahkan cek direktori contoh-app ya:
```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
		_, _ = fmt.Fprintf(writer, "Hello World!")
	})

	_ = http.ListenAndServe(":8080", nil)
}

```
Kemudian coba run dengan mengetikkan perintah, di terminal IDE atau terminal bawaan sistem operasi:
```go
go run main.go
```
Setelah itu buka browser, ketik:
```
localhost:8080
```
Jika berhasil pesan yang tampil adalah **Hello World**

Lanjut: silahkan membuat file dengan nama **Dockerfile** lalu ketik seperti di bawah ini:
```
#image yang udah ada local kita
FROM golang:alpine

#copy si main.go
COPY main.go /app/main.go

#cara running
CMD ["go", "run", "/app/main.go"]
```
Selanjutnya, kita build image nya dengan cara:
```
docker build --tag app-golang:1.0 .
```
Maksdunya --tag app-golang:1.0 adalah pemberian nama imagenya, kemudian . adalah direktori yang ada saat buat image.
Jika berhasil akan keluar informasi seperti ini:
```
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM golang:alpine
 ---> 53efefffaa70
Step 2/3 : COPY main.go /app/main.go
 ---> bde18b711ff4
Step 3/3 : CMD ["go", "run", "/app/main.go"]
 ---> Running in 9c25f3102f2c
Removing intermediate container 9c25f3102f2c
 ---> c9cb7bbcb1ec
Successfully built c9cb7bbcb1ec
Successfully tagged app-golang:1.0
```
Selanjutnya: kita cek apakah sudah ada image nya:
```
docker images
```
Jika sudah kita buat containernya:
```
docker container create --name app1 -p 8080:8080 app-golang:1.0
```
Jalankan container:
```
docker container start app1
```
Uji coba di browser kembali:
```
localhost:8080
```

#### Push Image Dari Dockerfile ke Registry (DockerHub)
1. Buka dan login di https://hub.docker.com/
2. Buat Repository baru, disini saya buat dengan nama `jutionck/app-golang` kalo kalian silahkan sesuaikan ya.
3. Login dulu (buka di terminal tadi) kemudian ketik ini:
    ```
   docker login
   ```
   Masukkan username dan password nya
4. Push image nya dengan cara seperti ini ya:
    Pertama ketik ini dulu:
    ```
   docker tag app-golang:1.0 jutionck/app-golang:1.0
   ```
   Baru ketik ini ya:
   ```
   docker push jutionck/app-golang:1.0
   ```
5. Selesai.

#### Environment Variable di Docker
1. Modifikasi main.go yang berada di folder contoh-app, menjadi:
    ```go
   package main
   
   import (
   	"log"
   	"net/http"
   	"os"
   )
   
   func main() {
   
   	port:= os.Getenv("PORT")
   	if port == "" {
   		log.Fatal("PORT env is required")
   	}
   
   	instanceID := os.Getenv("INSTANCE_ID")
   
   	mux := http.NewServeMux()
   	mux.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
   		if request.Method != "GET" {
   			http.Error(writer, "http not allowed", http.StatusBadGateway)
   			return
   		}
   
   		text := "Hello World"
   		if instanceID != "" {
   			text = text + ". From " + instanceID
   		}
   
   		_, _ = writer.Write([]byte(text))
   	})
   
   	server := new(http.Server)
   	server.Handler = mux
   	server.Addr = "0.0.0.0:" + port
   
   	log.Println("server starting at", server.Addr)
   	err := server.ListenAndServe()
   	if err != nil {
   		log.Fatal(err.Error())
   	}
   }
    ```
2. Modifikasi Dockerfile seperti ini:
    ```
   FROM golang:alpine
   
   RUN apk update && apk add --no-cache git
   
   WORKDIR /app
   
   COPY . .
   
   RUN go mod tidy
   
   RUN go build -o binary
   
   ENTRYPOINT ["/app/binary"]
   ```
3. Build Image dengan cara:
    ```
   docker build --tag app-golang-env:1.0 .
   ```
4. Buat container sekaligus setting env
    ```
   docker container create --name app2 -e PORT=8181 -e INSTANCE_ID="my first instance" -p 8181:8181 app-golang-env:1.0
   ```
5. Run container:
   ```
    docker container start app2
    ```

#### Integrasi Container dengan Network
Contoh, misal ketika kita membuat sebuah aplikasi kita mengkoneksikan 2 buah database, misal mongodb dan redis, jika kita menggunakan
Docker kedua database dan aplikasi kita di jadikan container, maka perlu adanya intergrasi agar
aplikasi dapat berjalan sebagaimana mestinya, disini saya tidak ada contoh aplikasi, tapi disini saya akan memberikan
contoh skrip nya saja, seperti ini (ini menggunakan aplikasi java):
```
docker container create --name my-java -p 8181:8181 -e MONGO_HOST=mongo -e MONGO_PORT=27017 -e REDIS_HOST=redis -e REDIS_PORT=6379 my-java:1.0
```

#### Docker Compose
Kenapa harus menggunakan fitur Docker Compose, bayangkan di atas, itu hanya menggunakan container 2 buah, jika kita mempunyai container yang lebih banyak maka repot. Maka solusinya
adalah kita menggunakan docker-compose. Tugasnya apa sih, docker compose ini sebagai automisasi proses-proses yang di atas.

1. Cara menggunakan nya adalah, silahkan buat file baru dengan nama `docker-compose.yml` kemudian silahkan cek di direktori `contoh-app` disitu saya sudah memberikan contoh isi nya
2. Untuk menjalankannya, ketikkan ini:
   ```
    docker-compose up -d
    ``` 
3. Untuk mematikannya (sekaligus menghapus containernya, jadi harus hati-hati dnegan perintah ini) :
    ```
   docker-compose down
   ```
4. Untuk stop:
    ```
   docker-compose stop
   ```
5. Untuk start:
    ```
   docker-compose start
   ```

#### Manage Data Docker
Ketika kita membuat sebuah database di docker dan kita melakukan operasi di dalamnya, tentu semua data pasti akan tersimpan di container tersebut.
Bagaimana jika kita tidak sengaja menghapus/container tiba-tiba crash, maka otomatis data yang di dalam nya akan rusak bahkan akan hilang.

Solusi dari masalah itu adalah, kita membuat sebuah volume bukan berada di di container, tetapi di dalam database itu sendiri.
Contoh pada database mongo itu berada pada `/data/db`

Untuk membuat sebuah volume ketik perintah berikut:
```
docker volume create namavolume
```

Jadi saat membuat container cukup tambahkan seperti ini:
```
docker container create --name mongo -p 8080:8080 -v namavolume:/data/db app-golang:1.0
```

#### Masuk kedalam Container
Contoh disini misal kita mau akses redis di client, cukup masuk seperti ini:
```
docker exec -t -i /bin/bash
```
```
type redis-cli
```
```
redis-cli
```

#### Menghapus Image yang tidak terpakai di Container
Ketikkan perintah:
```
docker system prune -a --volumes
```
