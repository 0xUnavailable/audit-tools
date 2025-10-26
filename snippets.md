# addressToBytes32
 function addressToBytes32(address) {
    const cleanAddress = address.toLowerCase().replace('0x', '');
    const bytes32 = '0x' + cleanAddress.padStart(64, '0');
    return bytes32;
}

# large number to BN
web3.utils.toBN(largeNumber);

