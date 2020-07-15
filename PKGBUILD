# Maintainer: acxz <akashpatel2008 at yahoo dot com>
# Contributor: Sven-Hendrik Haase <svenstaro@gmail.com>
# Contributor: Konstantin Gizdov (kgizdov) <arch@kge.pw>
# Contributor: Adria Arrufat (archdria) <adria.arrufat+AUR@protonmail.ch>
# Contributor: Thibault Lorrain (fredszaq) <fredszaq@gmail.com>

pkgbase=tensorflow-rocm
pkgname=(tensorflow-rocm tensorflow-opt-rocm python-tensorflow-rocm python-tensorflow-opt-rocm)
pkgver=2.2.0
_pkgver=2.2.0
pkgrel=1
pkgdesc="Library for computation using data flow graphs for scalable machine learning"
url="https://www.tensorflow.org/"
license=('APACHE')
arch=('x86_64')
depends=('c-ares' 'intel-mkl' 'onednn')
makedepends=('bazel' 'python-numpy' 'rocm' 'rccl' 'git' 'gcc'
             'python-pip' 'python-wheel' 'python-setuptools' 'python-h5py'
             'python-keras-applications' 'python-keras-preprocessing')
optdepends=('tensorboard: Tensorflow visualization toolkit'
            'python-pasta: tf_upgrade_v2 tool')
source=("$pkgname-$pkgver.tar.gz::https://github.com/tensorflow/tensorflow/archive/v${_pkgver}.tar.gz"
        build-against-actual-mkl.patch)
sha512sums=('94a2663497d333d543f363e9fea94fbcfdcdbbc0dfbaf009ba9181a808713aeb78f9b8805f56bea5fd3925a36f105427b3996795750589f378d8afbdadc2b86d'
            'e51e3f3dced121db3a09fbdaefd33555536095584b72a5eb6f302fa6fa68ab56ea45e8a847ec90ff4ba076db312c06f91ff672e08e95263c658526582494ce08')

get_pyver () {
  python -c 'import sys; print(str(sys.version_info[0]) + "." + str(sys.version_info[1]))'
}

check_dir() {
  if [ -d "${1}" ]; then
    return 0
  else
    >&2 echo Directory "${1}" does not exist or is a file! Exiting...
    exit 1
  fi
}

prepare() {
  # Allow any bazel version
  echo "*" > tensorflow-${_pkgver}/.bazelversion

  # Tensorflow actually wants to build against a slimmed down version of Intel MKL called MKLML
  # See https://github.com/intel/mkl-dnn/issues/102
  # MKLML version that Tensorflow wants to use is https://github.com/intel/mkl-dnn/releases/tag/v0.21
  patch -Np1 -d tensorflow-${_pkgver} -i "$srcdir"/build-against-actual-mkl.patch

  # Compile with C++17 by default (FS#65953)
  #sed -i "s/c++14/c++17/g" tensorflow-${_pkgver}/.bazelrc

  cp -r tensorflow-${_pkgver} tensorflow-${_pkgver}-rocm
  cp -r tensorflow-${_pkgver} tensorflow-${_pkgver}-opt-rocm

  # These environment variables influence the behavior of the configure call below.
  export PYTHON_BIN_PATH=/usr/bin/python
  export USE_DEFAULT_PYTHON_LIB_PATH=1
  export TF_NEED_JEMALLOC=1
  export TF_NEED_KAFKA=0
  export TF_NEED_OPENCL_SYCL=0
  export TF_NEED_AWS=0
  export TF_NEED_GCP=0
  export TF_NEED_HDFS=0
  export TF_NEED_S3=0
  export TF_ENABLE_XLA=1
  export TF_NEED_GDR=0
  export TF_NEED_VERBS=0
  export TF_NEED_OPENCL=0
  export TF_NEED_MPI=0
  export TF_NEED_TENSORRT=0
  export TF_NEED_NGRAPH=0
  export TF_NEED_IGNITE=0
  export TF_NEED_ROCM=1
  export TF_SET_ANDROID_WORKSPACE=0
  export TF_DOWNLOAD_CLANG=0
  export TF_NCCL_VERSION=2.6
  export TF_IGNORE_MAX_BAZEL_VERSION=1
  export TF_MKL_ROOT=/opt/intel/mkl
  export NCCL_INSTALL_PATH=/usr
  # export GCC_HOST_COMPILER_PATH=/usr/bin/gcc-9
  export GCC_HOST_COMPILER_PATH=/opt/rocm/hip/bin/hipcc
  # export HOST_C_COMPILER=/usr/bin/gcc-9
  export HOST_C_COMPILER=/opt/rocm/hip/bin/hipcc
  # export HOST_CXX_COMPILER=/usr/bin/g++-9
  export HOST_CXX_COMPILER=/opt/rocm/hip/bin/hipcc
  export TF_CUDA_CLANG=0  # Clang currently disabled because it's not compatible at the moment.
  export CLANG_CUDA_COMPILER_PATH=/usr/bin/clang
  export TF_CUDA_PATHS=/opt/cuda,/usr/lib,/usr
  export TF_CUDA_VERSION=$([ -e /opt/cuda ] && /opt/cuda/bin/nvcc --version | sed -n 's/^.*release \(.*\),.*/\1/p' || echo "")
  export TF_CUDNN_VERSION=$([ -e /usr/include/cudnn.h ] && sed -n 's/^#define CUDNN_MAJOR\s*\(.*\).*/\1/p' /usr/include/cudnn.h || echo "")
  export TF_CUDA_COMPUTE_CAPABILITIES=5.2,5.3,6.0,6.1,6.2,7.0,7.2,7.5,8.0

  export ROCM_PATH=/opt/rocm
  export HCC_HOME=$ROCM_PATH/hcc

  export HIP_PATH=$ROCM_PATH/hip
  export OPENCL_ROOT=$ROCM_PATH/opencl
  export PATH="$HCC_HOME/bin:$HIP_PATH/bin:${PATH}"
  export PATH="$ROCM_PATH/bin:${PATH}"
  export PATH="$OPENCL_ROOT/bin:${PATH}"
  export HCC_HOME=/opt/rocm/hcc
  export HIP_PATH=/opt/rocm/hip
  export PATH=$HCC_HOME/bin:$HIP_PATH/bin:$PATH

  # Required until https://github.com/tensorflow/tensorflow/issues/39467 is fixed.
  export CC=/opt/rocm/hip/bin/hipcc
  export CXX=/opt/rocm/hip/bin/hipcc
}

build() {
  echo "Building with rocm and without non-x86-64 optimizations"
  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  export CC_OPT_FLAGS="-march=x86-64"
  export TF_NEED_ROCM=1
  ./configure
  bazel \
    build -s --config=mkl -c opt \
      //tensorflow:libtensorflow.so \
      //tensorflow:libtensorflow_cc.so \
      //tensorflow:install_headers \
      //tensorflow/tools/pip_package:build_pip_package
  bazel-bin/tensorflow/tools/pip_package/build_pip_package --gpu "${srcdir}"/tmprocm


  echo "Building with rocm and with non-x86-64 optimizations"
  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  export CC_OPT_FLAGS="-march=haswell -O3"
  export TF_NEED_ROCM=1
  ./configure
  bazel \
    build -s --config=mkl --config=avx2_linux -c opt \
      //tensorflow:libtensorflow.so \
      //tensorflow:libtensorflow_cc.so \
      //tensorflow:install_headers \
      //tensorflow/tools/pip_package:build_pip_package
  bazel-bin/tensorflow/tools/pip_package/build_pip_package --gpu "${srcdir}"/tmpoptrocm
}

_package() {
  # install headers first
  install -d "${pkgdir}"/usr/include/tensorflow
  cp -r bazel-bin/tensorflow/include/* "${pkgdir}"/usr/include/tensorflow/
  # install python-version to get all extra headers
  WHEEL_PACKAGE=$(find "${srcdir}"/$1 -name "tensor*.whl")
  pip install --ignore-installed --upgrade --root "${pkgdir}"/ $WHEEL_PACKAGE --no-dependencies
  # move extra headers to correct location
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  find "${_srch_path}" -maxdepth 1 -mindepth 1 -type d -print0 | while read -rd $'\0' _folder; do
    cp -nr "${_folder}" "${pkgdir}"/usr/include/tensorflow/
  done
  # clean up unneeded files
  rm -rf "${pkgdir}"/usr/bin
  rm -rf "${pkgdir}"/usr/lib
  rm -rf "${pkgdir}"/usr/share

  # install the rest of tensorflow
  tensorflow/c/generate-pc.sh --prefix=/usr --version=${pkgver}
  sed -e 's@/include$@/include/tensorflow@' -i tensorflow.pc -i tensorflow_cc.pc
  install -Dm644 tensorflow.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow.pc
  install -Dm644 tensorflow_cc.pc "${pkgdir}"/usr/lib/pkgconfig/tensorflow_cc.pc
  install -Dm755 bazel-bin/tensorflow/libtensorflow.so "${pkgdir}"/usr/lib/libtensorflow.so.${pkgver}
  ln -s libtensorflow.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow.so.${pkgver:0:1}
  ln -s libtensorflow.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow.so
  install -Dm755 bazel-bin/tensorflow/libtensorflow_cc.so "${pkgdir}"/usr/lib/libtensorflow_cc.so.${pkgver}
  ln -s libtensorflow_cc.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow_cc.so.${pkgver:0:1}
  ln -s libtensorflow_cc.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow_cc.so
  install -Dm755 bazel-bin/tensorflow/libtensorflow_framework.so "${pkgdir}"/usr/lib/libtensorflow_framework.so.${pkgver}
  ln -s libtensorflow_framework.so.${pkgver} "${pkgdir}"/usr/lib/libtensorflow_framework.so.${pkgver:0:1}
  ln -s libtensorflow_framework.so.${pkgver:0:1} "${pkgdir}"/usr/lib/libtensorflow_framework.so
  install -Dm644 tensorflow/c/c_api.h "${pkgdir}"/usr/include/tensorflow/tensorflow/c/c_api.h
  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE
}

_python_package() {
  WHEEL_PACKAGE=$(find "${srcdir}"/$1 -name "tensor*.whl")
  pip install --ignore-installed --upgrade --root "${pkgdir}"/ $WHEEL_PACKAGE --no-dependencies

  # create symlinks to headers
  local _srch_path="${pkgdir}/usr/lib/python$(get_pyver)"/site-packages/tensorflow/include/
  check_dir "${_srch_path}"  # we need to quit on broken search paths
  find "${_srch_path}" -maxdepth 1 -mindepth 1 -type d -print0 | while read -rd $'\0' _folder; do
    rm -rf "${_folder}"
    _smlink="$(basename "${_folder}")"
    ln -s /usr/include/tensorflow/"${_smlink}" "${_srch_path}"
  done

  # tensorboard has been separated from upstream but they still install it with
  # tensorflow. I don't know what kind of sense that makes but we have to clean
  # it out from this pacakge.
  rm -rf "${pkgdir}"/usr/bin/tensorboard

  install -Dm644 LICENSE "${pkgdir}"/usr/share/licenses/${pkgname}/LICENSE
}

package_tensorflow-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM)"
  depends+=(rocm rccl)
  conflicts=(tensorflow)
  provides=(tensorflow)

  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  _package tmprocm
}

package_tensorflow-opt-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM and CPU optimizations)"
  depends+=(rocm rccl)
  conflicts=(tensorflow)
  provides=(tensorflow tensorflow-rocm)

  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  _package tmpoptrocm
}

package_python-tensorflow-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM)"
  depends+=(tensorflow-rocm python-termcolor python-astor python-gast python-numpy rocm python-protobuf absl-py rccl python-h5py python-keras-applications python-keras-preprocessing python-tensorflow-estimator python-opt_einsum python-astunparse)
  conflicts=(python-tensorflow)
  provides=(python-tensorflow)

  cd "${srcdir}"/tensorflow-${_pkgver}-rocm
  _python_package tmprocm
}

package_python-tensorflow-opt-rocm() {
  pkgdesc="Library for computation using data flow graphs for scalable machine learning (with ROCM and CPU optimizations)"
  depends+=(tensorflow-opt-rocm python-termcolor python-astor python-gast python-numpy rocm python-protobuf absl-py rccl python-h5py python-keras-applications python-keras-preprocessing python-tensorflow-estimator python-opt_einsum python-astunparse)
  conflicts=(python-tensorflow)
  provides=(python-tensorflow python-tensorflow-rocm)

  cd "${srcdir}"/tensorflow-${_pkgver}-opt-rocm
  _python_package tmpoptrocm
}

# vim:set ts=2 sw=2 et:
