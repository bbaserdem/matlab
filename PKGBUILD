#!/bin/bash

# Maintainer: Grey Christoforo <first name at last name dot net>
# Maintainer: Darcy Hu <hot123tea123@gmail.com>
# Maintainer: Grey Christoforo <first name at last name dot net>
# Contributor: Jingbei Li <i@jingbei.li>
# Maintainer: Antony Lee <anntzer dot lee at gmail dot com>

# PKGBUILD edited by; Batuhan Başerdem <lastname.firstname@gmail.com>
pkgname=matlab

## This PKGBUILD creates an Arch Linux package for the proprietary MATLAB application.
## In order to build the package the user must supply;
##      matlab.fik : Plain text file installation key
##      matlab.lic : The license file
##      matlab.tar : Software tarball

## GETTING LICENCE FILES:
##      Log into mathworks account; https://mathworks.com/myaccount/
##      From the licence, navigate to "Activate to Retrieve Licence File"
##      The File Installation Key will be available as plain text
##      Download the licence file

## GETTING TARBALL
##      Download the installer, unzip and run the installer
##      Set the -tmpdir flag to some directory
##      The installer will first install toolboxes in this directory
##      When the 'Downloading' switches to 'Installing', stop the installation
##      To be extra sure; pause, copy the temp directory, and then quit.
##      Merge the toolboxes with the files in the original zip.
##      The installer tarball should have the following structure;
# matlab
# ├── archives/                     # In zip and tmpdir
# ├── bin/                          # In zip
# ├── etc/                          # ???
# ├── help/                         # In zip
# ├── java/                         # In zip
# ├── licenses/                     # ???
# ├── sys/                          # In zip
# ├── ui/                           # In zip
# ├── activate.ini                  # In zip
# ├── install                       # In zip
# ├── installer_input.txt           # In zip
# ├── install_guide.pdf             # In zip
# ├── license_agreement.txt         # In zip
# ├── patents.txt                   # In zip
# ├── readme.txt                    # In zip
# ├── trademarks.txt                # In zip
# └── VersionInfo.xml               # In zip`

# To perform partial install, set to true and modify the products list

# Recently also added matlab engine for python
_partialinstall=false

pkgbase='matlab'
pkgname=('matlab' 'matlab-licenses' 'matlab-engine-for-python')
pkgver=9.6.0.1072779
pkgrel=3
pkgdesc='A high-level language for numerical computation and visualization'
arch=('x86_64')
url='http://www.mathworks.com'
license=(custom)
makedepends=('gendesk')
depends=('gcc6'
         'gconf'
         'gendesk'
         'glu'
         'gstreamer0.10-base'
         'gtk2'
         'libunwind'
         'libxp'
         'libxpm'
         'libxtst'
         'nss'
         'portaudio'
         'python2'
         'qt5-svg'
         'qt5-webkit'
         'qt5-websockets'
         'qt5-x11extras'
         'xerces-c')
source=("file://matlab.tar"
        "file://matlab.fik"
        "file://matlab.lic")
md5sums=("SKIP"
         "SKIP"
         "SKIP")

instdir="/opt/tmw/${pkgbase}"

prepare() {
    msg2 'Extracting file installation key'
    _fik=$(grep -o [0-9-]* ${pkgname}.fik)

    msg2 'Modifying the installer settings'
    sed -i "s,^# destinationFolder=,destinationFolder=${pkgdir}/${instdir},"    "${srcdir}/${pkgname}/installer_input.txt"
    sed -i "s,^# agreeToLicense=,agreeToLicense=yes,"                           "${srcdir}/${pkgname}/installer_input.txt"
    sed -i "s,^# mode=,mode=silent,"                                            "${srcdir}/${pkgname}/installer_input.txt"
    sed -i "s,^# fileInstallationKey=,fileInstallationKey=${_fik},"             "${srcdir}/${pkgname}/installer_input.txt"
    sed -i "s,^# licensePath=,licensePath=${srcdir}/matlab.lic,"                "${srcdir}/${pkgname}/installer_input.txt"

    msg2 'Creating desktop file'
    gendesk -f -n --pkgname "${pkgname}" --pkgdesc "${pkgdesc}" --categories "Development;Education;Science;Mathematics;IDE"
    sed -i "/^Exec=/ s,$, -desktop," "${srcdir}/${pkgname}.desktop"

    if [ ! -z ${_products+isSet} ]; then
        msg2 'Building a package with a subset of the licensed products.'
        for _product in "${_products[@]}"; do
              sed -i "/^#product.${_product}$/ s/^#//" "${srcdir}/${pkgname}/installer_input.txt"
        done
    fi
}

package_matlab() {
    msg2 'Starting MATLAB installer'
    "${srcdir}/${pkgname}/install" -inputFile "${srcdir}/${pkgname}/installer_input.txt"

    msg2 'Installing license'
    install -D -m644 "${pkgdir}/${instdir}/license_agreement.txt" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
    # Move licence files to the source files, so they can be installed with the side package
    rm -rf "${srcdir}/${pkgname}/licenses"
    mv "${pkgdir}/${instdir}/licenses" "${srcdir}/licenses"
    mkdir -p "${pkgdir}/${instdir}/licenses"

    msg2 'Creating links for executables'
    install -d -m755 "${pkgdir}/usr/bin/"
    for _executable in deploytool matlab mbuild mcc; do
        ln -s "${instdir}/bin/${_executable}" "${pkgdir}/usr/bin/${_executable}"
    done
    ln -s "${instdir}/bin/mex" "${pkgdir}/usr/bin/mex-$pkgbase"

    msg2 'Installing desktop files'
    install -D -m644 "${pkgname}.desktop" "${pkgdir}/usr/share/applications/${pkgname}.desktop"

    msg2 'Creating backup directories'
    mkdir -p "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts"
    mkdir -p "${pkgdir}/${instdir}/backup/sys/os/glnxa64"

    msg2 'Configuring mex options'
    cp "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gcc_glnxa64.xml" "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts/"
    sed -i "s/gcc/gcc-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gcc_glnxa64.xml"
    cp "${pkgdir}/${instdir}/bin/glnxa64/mexopts/g++_glnxa64.xml" "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts/"
    sed -i "s/g++/g++-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/g++_glnxa64.xml"
    cp "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran.xml" "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts/"
    sed -i "s/gfortran/gfortran-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran.xml"
    cp "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran6.xml" "${pkgdir}/${instdir}/backup/bin/glnxa64/mexopts/"
    sed -i "s/gfortran/gfortran-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran6.xml"
    sed -i "s/gfortran6-/gfortran-6/g" "${pkgdir}/${instdir}/bin/glnxa64/mexopts/gfortran6.xml"

    # https://bbs.archlinux.org/viewtopic.php?id=236821
    msg2 'Removing unused library files'
    # See $MATLABROOT/sys/os/glnxa64/README.libstdc++
    mv "${pkgdir}/${instdir}/sys/os/glnxa64/"{libstdc++.so.6.0.22,libstdc++.so.6,libgcc_s.so.1,libgfortran.so.3.0.0,libgfortran.so.3,libquadmath.so.0.0.0,libquadmath.so.0} "${pkgdir}/${instdir}/backup/sys/os/glnxa64"
    # https://bbs.archlinux.org/viewtopic.php?id=236821
    mv "${pkgdir}/${instdir}/bin/glnxa64/"libfreetype.* "${pkgdir}/${instdir}/backup/bin/glnxa64"
    # make sure MATLAB can find libgfortran.so.3
    cp "${pkgdir}/${instdir}/bin/matlab" "${pkgdir}/${instdir}/backup/bin"
    sed -i 's,LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`",LD_LIBRARY_PATH="`eval echo $LD_LIBRARY_PATH`:/usr/lib/gcc/x86_64-pc-linux-gnu/'$(pacman -Q gcc6 | awk '{print $2}' | cut -d- -f1)'",g' "${pkgdir}/${instdir}/bin/matlab"

    # Move external files such that they can be used to build matlab

}

package_matlab-licenses() {
    depends=("matlab=$pkgver")
    mkdir -p "${pkgdir}/${instdir}"
    mv "${srcdir}/licenses" "${pkgdir}/${instdir}/licenses"
}

package_matlab-engine-for-python() {
    depends=("matlab=$pkgver")
    mkdir -p "${pkgdir}/${instdir}"
    # I do not have the limitation of running setup.py in 'build()', as the
    #   package files neccessary are also built using this package.

    # https://aur.archlinux.org/packages/matlab-engine-for-python/
    # https://www.mathworks.com/help/matlab/matlab_external/install-the-matlab-engine-for-python.html
    msg2 'Installing MATLAB engine API for Python'
    cd "${pkgdir}/${instdir}/extern/engines/python"
    # We get the latest python supported version
    sys_python_version="$(python -c 'import sys; print(sys.version_info.minor)')"
    mat_supported_python="$(find . -name 'matlabengineforpython*.so' | sort | grep -Po '(?<=\d_)\d' | tail -1)"
    cat "import sys
sys.version_info = (3, ${mat_supported_python}, 0)" > "${srcdir}/sitecustomize.py"
    # Run the installation script, with the new sitecustomize.py
    PYTHONPATH="${srcdir}" python setup.py \
        build --build-base="${srcdir}" \
        install --root="${srcdir}" \
        --optimize=1

    # Fix version confusion
    if [[ "${sys_python_version}" != "${mat_supported_python}" ]] ; then
        # Copy files to usr/lib
        cp -r "${srcdir}/usr/lib/python3.${mat_supported_python}" "${sys_python_version}"
        # Fix the egg name in usr/lib
        _oldegg="$(ls "${srcdir}/usr/lib/python3.${sys_python_version}/site-packages/"*"-py3.${mat_supported_python}.egg-info")"
        mv "${_oldegg}" "${_oldegg%-py3."${mat_supported_python}".egg-info}-py3.${sys_python_version}.egg-info"
        # Patch the __init__.py
        sed -i "s/sys.version_info/(3, ${mat_supported_python}, 0)/" \
            "${srcdir}/usr/lib/python3.${sys_python_version}/site-packages/${pkgname}/engine/__init__.py"
    fi
}

if ${_partialinstall} && [ -z ${_products+isSet} ]; then
    _products=(
        "5G_Toolbox"
        "Aerospace_Blockset"
        "Aerospace_Toolbox"
        "Antenna_Toolbox"
        "Audio_System_Toolbox"
        "Automated_Driving_System_Toolbox"
        "Bioinformatics_Toolbox"
        "Communications_Toolbox"
        "Computer_Vision_System_Toolbox"
        "Control_System_Toolbox"
        "Curve_Fitting_Toolbox"
        "DO_Qualification_Kit"
        "DSP_System_Toolbox"
        "Data_Acquisition_Toolbox"
        "Database_Toolbox"
        "Datafeed_Toolbox"
        "Deep_Learning_Toolbox"
        "Econometrics_Toolbox"
        "Embedded_Coder"
        "Filter_Design_HDL_Coder"
        "Financial_Instruments_Toolbox"
        "Financial_Toolbox"
        "Fixed_Point_Designer"
        "Fuzzy_Logic_Toolbox"
        "GPU_Coder"
        "Global_Optimization_Toolbox"
        "HDL_Coder"
        "HDL_Verifier"
        "IEC_Certification_Kit"
        "Image_Acquisition_Toolbox"
        "Image_Processing_Toolbox"
        "Instrument_Control_Toolbox"
        "LTE_HDL_Toolbox"
        "LTE_Toolbox"
        "MATLAB"
        "MATLAB_Coder"
        "MATLAB_Compiler"
        "MATLAB_Compiler_SDK"
        "MATLAB_Distributed_Computing_Server"
        "MATLAB_Production_Server"
        "MATLAB_Report_Generator"
        "Mapping_Toolbox"
        "Model_Predictive_Control_Toolbox"
        "Model_Based_Calibration_Toolbox"
        "OPC_Toolbox"
        "Optimization_Toolbox"
        "Parallel_Computing_Toolbox"
        "Partial_Differential_Equation_Toolbox"
        "Phased_Array_System_Toolbox"
        "Polyspace_Bug_Finder"
        "Polyspace_Code_Prover"
        "Powertrain_Blockset"
        "Predictive_Maintenance_Toolbox"
        "RF_Blockset"
        "RF_Toolbox"
        "Risk_Management_Toolbox"
        "Robotics_System_Toolbox"
        "Robust_Control_Toolbox"
        "Sensor_Fusion_and_Tracking_Toolbox"
        "Signal_Processing_Toolbox"
        "SimBiology"
        "SimEvents"
        "Simscape"
        "Simscape_Driveline"
        "Simscape_Electrical"
        "Simscape_Fluids"
        "Simscape_Multibody"
        "Simulink"
        "Simulink_3D_Animation"
        "Simulink_Check"
        "Simulink_Code_Inspector"
        "Simulink_Coder"
        "Simulink_Control_Design"
        "Simulink_Coverage"
        "Simulink_Design_Optimization"
        "Simulink_Design_Verifier"
        "Simulink_Desktop_Real_Time"
        "Simulink_PLC_Coder"
        "Simulink_Real_Time"
        "Simulink_Report_Generator"
        "Simulink_Requirements"
        "Simulink_Test"
        "Spreadsheet_Link"
        "Stateflow"
        "Statistics_and_Machine_Learning_Toolbox"
        "Symbolic_Math_Toolbox"
        "System_Identification_Toolbox"
        "Text_Analytics_Toolbox"
        "Trading_Toolbox"
        "Vehicle_Dynamics_Blockset"
        "Vehicle_Network_Toolbox"
        "Vision_HDL_Toolbox"
        "WLAN_Toolbox"
        "Wavelet_Toolbox"
        )
fi
