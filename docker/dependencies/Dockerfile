# docker build --build-arg ROCQ_TAG=8.16 -t smtcoq/smtcoq:latest-coq-8.16 .
# docker run -it smtcoq/smtcoq:latest-coq-8.16
# docker push smtcoq/smtcoq:latest-coq-8.16
# docker run -w /home/rocq/smtcoq -v "$(pwd):/home/rocq/smtcoq" -it smtcoq/smtcoq:latest-coq-8.16

############################################################
# Dockerfile to build SMTCoq dependencies
############################################################

ARG ROCQ_TAG="dev"
FROM rocq/rocq-prover:${ROCQ_TAG}

USER rocq

################### SYSTEM PREREQUISITES ###################

RUN sudo apt-get -y update && \
    sudo apt-get -y install wget atool bison flex

#################### OPAM PREREQUISITES ####################

WORKDIR /home/rocq

RUN opam update && \
    opam upgrade -y && \
    opam install -y num

###################### COMPILE ZChaff ######################

WORKDIR /home/rocq

RUN wget http://www.princeton.edu/~chaff/zchaff/zchaff.2008.10.12.zip && \
    aunpack zchaff.2008.10.12.zip && \
    cd zchaff && \
    sed -i -re 's/\/\/ #define VERIFY_ON/#define VERIFY_ON/' zchaff_solver.cpp && \
    make

###################### COMPILE veriT #######################

WORKDIR /home/rocq

RUN wget https://usr.lmf.cnrs.fr/~ckeller/Documents-recherche/Smtcoq/veriT9f48a98.tar.gz && \
    aunpack veriT9f48a98.tar.gz && \
    cd veriT9f48a98 && \
    sed -i -e '3i#include "DAG-print.h"' src/instantiation/ccfv-{constr,mod}.c && \
    sed -i -e '4iconst char *byte_to_binary(unsigned x, unsigned size);' src/instantiation/ccfv-mod.c && \
    sed -i -e '248iTproof proof_tmp_eq_norm(Tproof clause, TDAG DAG);' src/proof/proof.h && \
    sed -i -e 's/bb_data/bb_data_xx/g' src/arith/LA-xx.c.tpl && \
    sed -i -e 's/conflict_eqs/conflict_eqs_xx/g' src/arith/LA-xx.c.tpl && \
    sed -i -e 's/conflict_lits/conflict_lits_xx/g' src/arith/LA-xx.c.tpl && \
    sed -i -e 's/LA_bound_ranking/LA_bound_ranking_xx/g' src/arith/LA-xx.c.tpl && \
    sed -i -e 's/veriT_conflict_eqs_xx/veriT_conflict_eqs/g' src/arith/LA-xx.c.tpl && \
    sed -i -e '200iextern' src/arith/simplex-xx.c.tpl && \
    sed -i -e '3iTstack_simplex_var integer_stack;' src/arith/simplex.c && \
    sed -i -e '2350d' src/congruence/congruence.c && \
    sed -i -e '28,32d' src/instantiation/inst-man.c && \
    sed -i -e '20iextern' src/instantiation/inst-trigger.c && \
    sed -i -e '403iextern' src/SAT/veriT-SAT.h && \
    sed -i -e '29d' src/parsers/tptp/tptplex.l && \
    autoconf && \
    ./configure && \
    make

###################### DOWNLOAD CVC4 #######################

WORKDIR /home/rocq

RUN wget http://cvc4.cs.stanford.edu/downloads/builds/x86_64-linux-opt/cvc4-1.6-x86_64-linux-opt && \
    chmod ugo+x cvc4-1.6-x86_64-linux-opt

##################### LFSC SIGNATURES ######################

WORKDIR /home/rocq

RUN git clone https://github.com/smtcoq/smtcoq.git smtcoq-sig

ENV LFSCSIGS="/home/rocq/smtcoq-sig/src/lfsc/tests/signatures/"

ENV DONTSHOWVERIT="yes"

###################### DOWNLOAD cvc5 #######################

WORKDIR /home/rocq

RUN wget https://github.com/cvc5/cvc5/releases/download/cvc5-1.0.7/cvc5-Linux && \
    chmod ugo+x cvc5-Linux

######################### BINARIES #########################

WORKDIR /home/rocq/bin

RUN ln -s /home/rocq/zchaff/zchaff && \
    ln -s /home/rocq/veriT9f48a98/veriT && \
    ln -s /home/rocq/cvc4-1.6-x86_64-linux-opt cvc4 && \
    ln -s /home/rocq/cvc5-Linux cvc5

ENV PATH="/home/rocq/bin:${PATH}"

WORKDIR /home/rocq
