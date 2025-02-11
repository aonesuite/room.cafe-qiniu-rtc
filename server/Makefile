install:
	cd src/vendor/github.com/cespare/reflex && go install

# 代码风格检验
fix:
	CGO_ENABLED=0 go install -v ./src/vendor/github.com/qiniu/checkstyle/gocheckstyle/...
	gocheckstyle -config=scripts/golangstyle.json -reporter=plain src 2>&1 | tee -a fix.log

	CGO_ENABLED=0 go install -v ./src/vendor/golang.org/x/lint/golint/...
	golint $$(go list ./... | grep -v /vendor/) | tee -a fix.log

# 构建
build:
	rm -rf pkg bin/room.cafe
	go clean -i ./src/room.cafe/...
	CGO_ENABLED=0 go install -v ./src/room.cafe/...

# 运行后端服务
serve: install
	cd src && reflex -s -R '^vendor/' -R '_test.go$$'\
		-- go run room.cafe/*.go

# 测试
test:
	cd src; CGO_ENABLED=0 go test -v $$(go list ./... | grep -v vendor)

# 测试报告
report:
	CGO_ENABLED=0 go install -v ./src/vendor/github.com/axw/gocov/gocov
	CGO_ENABLED=0 go install -v ./src/vendor/github.com/AlekSi/gocov-xml
	CGO_ENABLED=0 go install -v ./src/vendor/gopkg.in/matm/v1/gocov-html
	rm -rf coverage.html coverage.xml coverage.txt coverage.log
	sh ./scripts/coverage.sh --html --xml | tee coverage.log

# 编译 linux 64位 运行二进制
build-linux-64:
	rm -rf pkg bin
	go clean -i ./src/room.cafe/...
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go install -v ./src/room.cafe/...

