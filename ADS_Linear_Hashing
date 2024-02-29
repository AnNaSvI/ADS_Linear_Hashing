#ifndef ADS_SET_H
#define ADS_SET_H

template<typename Key, size_t N = 7>
class ADS_set {
public:
    class Iterator;
    using value_type = Key;
    using key_type = Key;
    using reference = value_type &;
    using const_reference = const value_type &;
    using size_type = size_t;
    using difference_type = std::ptrdiff_t;
    using const_iterator = Iterator;
    using iterator = Iterator;
    using key_equal = std::equal_to<key_type>; // Hashing
    using hasher = std::hash<key_type>;        // Hashing
private:
 static constexpr size_t INITIAL_BUCKET_COUNT = 8;
    struct Bucket {
        size_t next_empty;
        key_type *bucket_values;
        Bucket *next_bucket;

        Bucket(): next_empty(0), bucket_values(new key_type[N]), next_bucket(nullptr){}

        void insert_in_bucket(const key_type &data) {
            this->bucket_values[next_empty] = data;
            next_empty++;

        }

        size_type contains(const key_type &data) {
          for (size_type i = 0; i < next_empty; ++i) {
            if (key_equal{}(bucket_values[i], data)) {
             return i; 
            }
          }
        return next_empty;
       }

    };

    size_type next_to_split; 
    size_type size_table;
    size_type next_empty;
    Bucket **array_bucket;
    int counter;  
    size_type split_index; 
public:
    ADS_set() {
        counter = 0;
        size_table = INITIAL_BUCKET_COUNT;
        array_bucket = new Bucket *[size_table];
        next_empty = 1;
        array_bucket[0] = new Bucket();
        split_index = 0;
        next_to_split = 0;
    }

  

    ADS_set(std::initializer_list<key_type> ilist) : ADS_set() {
        insert(ilist);
    }

    template<typename InputIt>
    ADS_set(InputIt first, InputIt last): ADS_set() {
            insert(first, last);
    }

    ADS_set(const ADS_set &other) : ADS_set() {
        for (auto i = other.begin(); i != other.end(); i++) {
            insert(*i);
        }
    } 
   
   ~ADS_set() {
    for (size_type i = 0; i < next_empty; ++i) {
        Bucket* current_bucket = array_bucket[i];
        while (current_bucket != nullptr) {
            Bucket* temp = current_bucket->next_bucket;
            delete[] current_bucket->bucket_values;
            delete current_bucket;
            current_bucket = temp;
        }
    }
    delete[] array_bucket;
}


    size_type h1(const size_type &key) const {  
        return key % (1 << split_index);
    }

    size_type h2(const size_type &key) const {          
        return key % (1 << (split_index + 1));
    }

    size_type size() const {
        return counter;
    }

    bool empty() const {
        return (size() == 0);
    }

    void clear() {
        ADS_set tmp;
        swap(tmp);
    }

    ADS_set &operator=(const ADS_set &other) {

        if(this != &other){
            clear();
          for (auto i = other.begin(); i != other.end(); i++) {
            insert(*i);
        }
        return *this;
        }
        return *this;
    } 
    ADS_set &operator=(std::initializer_list<key_type> ilist) {
       clear(); 
       insert(ilist); 
       return *this;

    } 

    

    std::pair<Bucket *, bool> first_empty(Bucket *b) {
        for (; b != nullptr; b = b->next_bucket) {
          if (!(b->next_empty == N)) {
             return {b, true}; 
            } else if (b->next_bucket == nullptr) {
                 return {b, false}; 
                }     
            }
        return {nullptr, false};  
    }

    std::pair<const_iterator, bool> insert(const key_type &key) {
         std::pair<const_iterator, size_type> r = find_peair(key);
        if (r.first == this->end()) {
            auto first_emty = first_empty(*(array_bucket + r.second)); 
            if (first_emty.second == false) {
                first_emty.first->next_bucket = new Bucket();
                first_emty.first->next_bucket->insert_in_bucket(key);
                counter++;
                if (next_empty == size_table) {
                     Bucket **newArray = new Bucket *[size_table * 2];
                    std::copy(array_bucket, array_bucket + next_empty, newArray);
                    delete[] array_bucket;
                    array_bucket = newArray;
                    size_table = size_table * 2;
                        }
                array_bucket[next_empty++] = new Bucket();

                bool flag = element_ersetzen(next_to_split);
                if(flag){
                   bool overflow = false; 

                for (Bucket *i = array_bucket[next_to_split]; i != nullptr;) {
                    if (i->next_empty == 0 && !overflow) {
                        overflow = true;
                    }

                    if (i->next_bucket != nullptr && i->next_bucket->next_empty == 0 && overflow) { 
                        auto ptr = i->next_bucket->next_bucket;
                        delete[] i->next_bucket->bucket_values;
                        delete i->next_bucket;
                        i->next_bucket = nullptr;
                        i->next_bucket = ptr;
                    } else {
                        i = i->next_bucket;
                    }
                }
                        }
                 if (next_to_split ==static_cast<size_type>(1 << split_index)) {
                     split_index++;
                    next_to_split = 0;
                    } else {
                        next_to_split++;
                        }
                return std::make_pair(find(key), true);
            } else {
                first_emty.first->insert_in_bucket(key);

                counter++;
                return std::make_pair(const_iterator(next_empty, array_bucket, r.second, first_emty.first, first_emty.first->next_empty - 1), true);
            }
          
        }
        return std::make_pair(r.first, false);
    } 

    
    void insert(std::initializer_list<key_type> ilist) {
        for (const auto &i: ilist) {
            insert(i);
        }
    }

    template<typename InputIt>
    void insert(InputIt first, InputIt last) {
        for (auto i= first; i != last; i++) {
            insert(*i);
        }
    }


    size_type count(const key_type &key) const {
       size_type r = h1(hasher{}(key));
        if (r < next_to_split) {
         r = h2(hasher{}(key));
        }

        for (Bucket *i = array_bucket[r]; i != nullptr; i = i->next_bucket) {
        size_type index = i->contains(key);
        if (index != i->next_empty) {
                return 1;
        }
        }

        return 0;    
    }

    iterator find(const key_type &key) const {
     size_type hesh = h1(hasher{}(key));
        if (hesh < next_to_split) {
            hesh = h2(hasher{}(key));
        }
        for (Bucket *i = array_bucket[hesh]; i != nullptr; i = i->next_bucket) {
            size_type index = i->contains(key);
            if (index != i->next_empty)
                return iterator(next_empty, array_bucket, hesh, i, index);
        }
        return this->end();
    }

    std::pair<const_iterator, size_type> find_peair(const key_type &key) const {
       size_type r = h1(hasher{}(key));
        if (r < next_to_split) {
            r = h2(hasher{}(key));
        }

        for (Bucket* bucket = array_bucket[r]; bucket != nullptr; bucket = bucket->next_bucket) {
            size_type index = bucket->contains(key);
            if (index != bucket->next_empty) {
                return {const_iterator(next_empty, array_bucket, r, bucket, index), r};
            }
        }

        return {this->end(), r};  
    }

   
  
    size_type erase(const key_type &key) {
         size_type row = h1(hasher{}(key));
    if (row < next_to_split) {
        row = h2(hasher{}(key));
    }
    for (Bucket *i = array_bucket[row]; i != nullptr; i = i->next_bucket) {
        size_type index = 0;
        for (; index < i->next_empty; ++index) {
            if (key_equal{}(i->bucket_values[index], key)) {
                break; 
            }
        }

        if (index != i->next_empty) {
            
            for (size_type j = index; j < i->next_empty - 1; ++j) {
                i->bucket_values[j] = i->bucket_values[j + 1];
            }
            --(i->next_empty); 
            counter--;
            
            bool overflow = false; 

        for (Bucket *i = array_bucket[row]; i != nullptr;) {
            if (i->next_empty == 0 && !overflow) {
                overflow = true;
            }

            if (i->next_bucket != nullptr && i->next_bucket->next_empty == 0 && overflow) { 
                auto ptr = i->next_bucket->next_bucket;
                delete[] i->next_bucket->bucket_values;
                delete i->next_bucket;
                i->next_bucket = nullptr;
                i->next_bucket = ptr;
            } else {
                i = i->next_bucket;
            }
        }
           
            return 1; 
        }
    }
    return 0; 
    }

   

    bool element_ersetzen(const size_type &index) {
       bool flag = false;
            for (Bucket *i = *(array_bucket + index); i != nullptr; i = i->next_bucket) {
            for (size_type key = 0; key < i->next_empty; key++) {
                size_type index2 = h2(hasher{}(i->bucket_values[key]));
                if (index2 != index) { 
                    auto first_emty = first_empty(*(array_bucket + index2));

                    if (first_emty.second == false) {
                        first_emty.first->next_bucket = new Bucket();
                        first_emty.first->next_bucket->insert_in_bucket(i->bucket_values[key]);
                    } else {
                        first_emty.first->insert_in_bucket(i->bucket_values[key]);
                    }


                    size_type index_loschen;
                    for (index_loschen=0 ; index_loschen < i->next_empty; ++index_loschen) {
                        if (key_equal{}(i->bucket_values[index_loschen], i->bucket_values[key])) {
                            break; 
                        }
                    }

                    if (index_loschen != i->next_empty) {
                       
                        for (size_type j = index_loschen; j < i->next_empty - 1; ++j) {
                            i->bucket_values[j] = i->bucket_values[j + 1];
                        }
                        --(i->next_empty); 
                    }

                    flag = true;

                    if (i->next_empty == 0) {
                        break;
                    } else {
                        key--;
                    }
                }
            }
        }

    return flag;
}


    void swap(ADS_set &other) {
        std::swap(this->array_bucket, other.array_bucket);
        std::swap(split_index, other.split_index);
        std::swap(next_to_split, other.next_to_split);
        std::swap(counter, other.counter);
        std::swap(next_empty, other.next_empty);
        std::swap(size_table, other.size_table);
        

    }
    friend bool operator==(const ADS_set<Key, N> &lhs, const ADS_set<Key, N> &rhs) {
        if (lhs.size() != rhs.size()) {
            return false;
        }
        for (const auto& element : rhs) {
            if (lhs.count(element) == 0) {
                return false;
            }
        }
        return true;
    }

    friend bool operator!=(const ADS_set<Key, N> &lhs, const ADS_set<Key, N> &rhs) {
        return !(lhs == rhs);
    }

    void dump(std::ostream &o = std::cerr) const {
    o << "ADS_set contents:\n";
    for (size_type i = 0; i < next_empty; ++i) {
        o << "Bucket " << i << ":\n";
        Bucket *current_bucket = array_bucket[i];
        while (current_bucket != nullptr) {
            o << "  ";
            for (size_type j = 0; j < current_bucket->next_empty; ++j) {
                o << current_bucket->bucket_values[j] << " ";
            }
            o << "\n";
            current_bucket = current_bucket->next_bucket;
        }
    }
}


    const_iterator begin() const {
        size_type i = 0;
        while (i < next_empty) {
            Bucket* j = array_bucket[i];
            while (j != nullptr) {
                if (j->next_empty != 0) {
                    return const_iterator(next_empty, array_bucket, i, j, 0);
                }
                j = j->next_bucket;
            }
            ++i;
        }
        return this->end();
    }


    const_iterator end() const {
        return const_iterator(nullptr);
    } 

};

template<typename Key, size_t N>
class ADS_set<Key, N>::Iterator {
public:
    using iterator_category = std::forward_iterator_tag;
    using value_type = ADS_set::value_type;
    using difference_type = ADS_set::difference_type;
    using pointer = const ADS_set::value_type *;
    using reference = ADS_set::const_reference;

private:
    pointer ptr;
    size_type index_latzt;
    Bucket **array_bucket;
    size_type index;
    Bucket *index_bucket;
    size_type index_in_bucket;
    
public:
    Iterator() {
        ptr = nullptr;
        index = 0;
        index_latzt = 0;
        index_in_bucket = 0;
        array_bucket = nullptr;
        index_bucket = nullptr;
    }

    Iterator(size_type index_latzt, Bucket **array_bucket, size_type index, Bucket *index_bucket, size_type index_in_bucket)
    : index_latzt(index_latzt), array_bucket(array_bucket), index(index), index_bucket(index_bucket), index_in_bucket(index_in_bucket) {
        
    if (index_bucket != nullptr && index_bucket->bucket_values != nullptr && index_in_bucket < index_bucket->next_empty) {
        ptr = index_bucket->bucket_values + index_in_bucket;
    } else {
        ptr = nullptr;
    }
}


Iterator(pointer ptr): ptr(ptr){
    index=0;
    index_latzt=0;
    index_in_bucket=0;
    array_bucket=nullptr;
    index_bucket=nullptr;
}

     Iterator &operator++() {
    index_in_bucket++;
    while (true) {
        if (index == index_latzt || (index_bucket == nullptr && index_in_bucket >= index_latzt)) {
            ptr = nullptr;
            return *this;
        }

        if (index_bucket == nullptr || index_in_bucket >= index_bucket->next_empty) {
            if (index_bucket == nullptr || index_in_bucket == index_bucket->next_empty) {
                index_bucket = (index_bucket != nullptr) ? index_bucket->next_bucket : nullptr;
                index_in_bucket = 0;
            }

            if (index_bucket == nullptr) {
                index++;
                if (index < index_latzt) {
                    index_bucket = array_bucket[index];
                }
            }
        } else {
            ptr = index_bucket->bucket_values + index_in_bucket;
            return *this;
        }
    }

}

    Iterator operator++(int) {
        auto temp = *this;
        ++*this; 
        return temp;
    }


    reference operator*() const {
        return *ptr;
    }

    pointer operator->() const {
        return ptr;
    }

    friend bool operator==(const Iterator &lhs, const Iterator &rhs) {
        if(lhs.ptr != rhs.ptr) {
            return false;
        } else {
            return lhs.ptr == rhs.ptr;
        }
    }
    friend bool operator!=(const Iterator &lhs, const Iterator &rhs) { return !(lhs == rhs); }

    

   
}; 

template <typename Key, size_t N> void swap(ADS_set<Key,N> &lhs, ADS_set<Key,N> &rhs) {
    lhs.swap(rhs);
}


#endif
