default : all

%.cfg :
	./TAppEncoderStatic -c ../cfg/encoder_intra_main.cfg -c ../cfg/per-sequence/$*.cfg --FramesToBeEncoded=50 -q 22  > $*_QP22.out
	rm rec.yuv str.bin
	./TAppEncoderStatic -c ../cfg/encoder_intra_main.cfg -c ../cfg/per-sequence/$*.cfg --FramesToBeEncoded=50 -q 27 > $*_QP27.out
	rm rec.yuv str.bin
	./TAppEncoderStatic -c ../cfg/encoder_intra_main.cfg -c ../cfg/per-sequence/$*.cfg --FramesToBeEncoded=50 -q 32 > $*_QP32.out
	rm rec.yuv str.bin
	./TAppEncoderStatic -c ../cfg/encoder_intra_main.cfg -c ../cfg/per-sequence/$*.cfg --FramesToBeEncoded=50 -q 37 > $*_QP37.out
	rm rec.yuv str.bin
	echo TERMINADO $@

all : BasketballPass.cfg \
BQMall.cfg \
FourPeople.cfg \
Johnny.cfg \
Kimono.cfg \
ParkScene.cfg \
PartyScene.cfg \
PeopleOnStreet.cfg \
RaceHorsesC.cfg \
Traffic.cfg

