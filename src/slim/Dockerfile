FROM gcc AS build

COPY main.cpp .

RUN g++ main.cpp -o main -static

FROM scratch

COPY --from=build main .

CMD ["./main"]