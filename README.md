# Diamond-Pattern
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract DiamondStorage {
    struct Facet {
        address facetAddress;
        bytes4[] functionSelectors;
    }
    mapping(bytes4 => address) public selectorToFacet;
}

contract Diamond is DiamondStorage {
    function addFunction(bytes4 _selector, address _facet) external {
        selectorToFacet[_selector] = _facet;
    }

    fallback() external payable {
        address facet = selectorToFacet[msg.sig];
        require(facet != address(0), "Function not found");
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
